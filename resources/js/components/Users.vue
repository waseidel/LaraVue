<template>
    <div class="container">
        <div class="row mt-5">
          <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users Table</h3>
                <div class="card-tools">
                    <button class="btn btn-success" @click="newModal">Add New <i class="fas fa-user-plus fa-fw"></i></button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <tbody><tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Email</th>
                    <th>Type</th>
                    <th>Registered At</th>
                    <th>Modify</th>
                  </tr>

                  <tr v-for="user in users" :key="user.id">
                    <td>{{user.id}}</td>
                    <td>{{user.name }}</td>
                    <td>{{user.email}}</td>
                    <td>{{user.type | capitalize}}</td>
                    <td>{{user.created_at | myDate}}</td>
                    <td>
                        <a href="#" @click="editModal(user)">
                            <i class="fa fa-edit blue"></i>
                        </a>
                        /
                        <a href="#" @click="deleteUser(user.id)">
                            <i class="fa fa-trash red"></i>
                        </a>
                    </td>
                  </tr>
                  
                </tbody></table>
              </div><!-- /.card-body -->
            </div><!-- /.card -->
          </div><!-- /.col-md-12 -->
        </div><!-- /.row -->

        <!-- Modal -->
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNewLabel" aria-hidden="true">
          <div class="modal-dialog modal-dialog-centered" role="document">
            <div class="modal-content">
              <div class="modal-header">
                <h5 v-show="!editmode" class="modal-title" id="addNewLabel">Add New</h5>
                <h5 v-show="editmode" class="modal-title" id="addNewLabel">Update User</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                  <span aria-hidden="true">&times;</span>
                </button>
              </div>
              <form @submit.prevent="editmode ? updateUser() : createUser()">
              <div class="modal-body">
                <!-- FORM -->
                  <!-- NAME -->
                <div class="form-group">
                  <input v-model="form.name" type="text" name="name" placeholder="Name" 
                    class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                  <has-error :form="form" field="name"></has-error>
                </div>
                  <!-- EMAIL -->
                <div class="form-group">
                  <input v-model="form.email" type="email" name="email" placeholder="Email Address" 
                    class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                  <has-error :form="form" field="email"></has-error>
                </div>
                  <!-- BIO -->
                <div class="form-group">
                  <textarea v-model="form.bio" name="bio" placeholder="Short bio for user (Optional)" class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                  <has-error :form="form" field="bio"></has-error>
                </div>
                  <!-- TYPE -->
                <div class="form-group">
                  <select name="type" v-model="form.type" class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                    <option value="">Select User Role</option>
                    <option value="admin">Admin</option>
                    <option value="user">Standard User</option>
                    <option value="author">Author</option>
                  </select>
                  <has-error :form="form" field="type"></has-error>
                </div>
                  <!-- NAME -->
                <div class="form-group">
                  <input v-model="form.password" type="password" name="password" placeholder="Password" 
                    class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                  <has-error :form="form" field="password"></has-error>
                </div>
                <!-- /FORM -->
              </div>
              <div class="modal-footer">
                <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                <button v-show="editmode" type="submit" class="btn btn-success">Update</button>
                <button v-show="!editmode" type="submit" class="btn btn-primary">Create</button>
              </div>
              </form>
            </div>
          </div>
        </div> <!--/modal -->

    </div><!-- /.container -->
</template>

<script>
    export default {
        data(){
          return {
            editmode: false,
            users: {},
            // Create a new form instance
            form: new Form({
              id: '',
              name : '',
              email: '',
              password: '',
              type: '',
              bio: '',
              photo: ''
            })
          }
        },
        methods: {
          updateUser(){
            this.$Progress.start();
            this.form.put('api/user/'+this.form.id)
            .then(()=>{
              Fire.$emit('AtualizaLista');
              $('#addNew').modal('hide');
              toast({
                type: 'success',
                title: 'User Updated in successfully'
              });
              this.$Progress.finish();
            })
            .catch(()=>{
              toast({
                type: 'error',
                title: 'Erro Ao Editar'
              })
              this.$Progress.fail();
            });
          },
          editModal(user){
            this.editmode = true;
            this.form.reset();
            $('#addNew').modal('show');
            this.form.fill(user);
          },
          newModal(){
            this.editmode = false;
            this.form.reset();
            $('#addNew').modal('show');
          },
          deleteUser(id){
            Swal({
              title: 'Are you sure?',
              text: "You won't be able to revert this!",
              type: 'warning',
              showCancelButton: true,
              confirmButtonColor: '#3085d6',
              cancelButtonColor: '#d33',
              confirmButtonText: 'Yes, delete it!'
            }).then((result) => {

              if (result.value) {
                //Send request to the serve
                this.form.delete('api/user/'+id).then(()=>{
                  Fire.$emit('AtualizaLista');
                    Swal(
                      'Deleted!',
                      'Your file has been deleted.',
                      'success'
                    )
                }).catch(()=>{
                  Swal("Failed!","There was something wronge.","warning");
                });
              }
              
            });

          },
          loadUsers(){
            axios.get("api/user").then(({ data }) => (this.users = data.data));
          },
          createUser(){
            this.$Progress.start();
            this.form.post('api/user')
            //Sucesso
            .then(()=>{
              Fire.$emit('AtualizaLista');
              $('#addNew').modal('hide');
              toast({
                type: 'success',
                title: 'User Created in successfully'
              });
              this.$Progress.finish();

            })
            //Falha
            .catch(()=>{
              this.$Progress.fail();
              toast({
                type: 'error',
                title: 'Erro Ao criar'
              })
              
            })


            
          }
        },
        created() {
            this.loadUsers();
            Fire.$on('AtualizaLista',() => {
              this.loadUsers();
            });
            //setInterval(() => this.loadUsers(),3000);
        }
    };
</script>
