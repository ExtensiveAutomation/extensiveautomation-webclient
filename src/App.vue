<template>
  <v-app >
    
    <navigation 
        :nav="nav"  >
    </navigation>
    <toolbar 
          @hideNavigation="hideNavigation" 
          @userProfile="userProfile"
          @userLogout="userLogout"
          > 
    </toolbar>

    
    <v-content>
      <router-view></router-view>
    </v-content>

    <v-footer class="pa-3">
        <v-spacer></v-spacer>
        <div>Extensive Automation &copy; - 2010-2019 - Denis MACHARD</div>
      </v-footer>
      
    <v-dialog v-model="dialog" persistent max-width="290">
      <v-card>
        <v-card-title>
            <span class="headline">Error</span>
          </v-card-title>
        <v-divider></v-divider>
        <v-card-text>{{ msg }}</v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="red darken-1" flat @click="closeError">OK</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="dialog_warning" persistent max-width="290">
      <v-card>
        <v-card-title>
            <span class="headline">Warning</span>
          </v-card-title>
        <v-divider></v-divider>
        <v-card-text>{{ msg }}</v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="red darken-1" flat @click="closeWarning">OK</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog  v-model="dialog_profile" 
          max-width="600px"
          persistent 
          no-click-animation
          transition=""
        >
        <v-card>
          <v-card-title>
            <span class="headline">User Profile</span>
            <v-spacer></v-spacer>
            <v-menu bottom left>
              <template v-slot:activator="{ on }">
                <v-btn icon v-on="on"  >
                  <v-icon>more_vert</v-icon>
                </v-btn>
              </template>
              <v-list>
                <v-list-tile @click="updatePassword">
                  <v-list-tile-title>Update password</v-list-tile-title>
                </v-list-tile>
              </v-list>
            </v-menu>

          </v-card-title>

          <v-divider></v-divider>
          <v-card-text>

            <v-container grid-list-md>
            <v-layout wrap>
            <v-flex  xs12>
              <v-text-field 
                  label="Login"
                  v-model="userprofile.api_login"
                  readonly
              ></v-text-field>
              <v-text-field 
                  label="Email"
                  v-model="userprofile.email"
                  readonly
              ></v-text-field>
              <v-text-field 
                  label="Api KEY"
                  v-model="userprofile.api_secret"
                  readonly
              ></v-text-field>
              <v-text-field
                    v-model="userprofile.level"
                    label="Level"
                    readonly
                  ></v-text-field>
              <v-text-field 
                  label="Default Project"
                  v-model="userprofile.default_project.name"
                  readonly
              ></v-text-field>
              <v-text-field 
                  label="Granted Project(s)"
                  v-model="projects_list"
                  readonly
              ></v-text-field>
              </v-flex>
                </v-layout>
              </v-container>
          </v-card-text>

          
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" flat 
                  @click="closeProfile"
                  :loading="loader">Close</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

    <v-dialog  v-model="dialog_password" 
          max-width="600px"
          persistent 
          no-click-animation
          transition=""
        >
        <v-card>
          <v-card-title>
            <span class="headline">User Password</span>
          </v-card-title>

          <v-divider></v-divider>
          <v-card-text>

            <v-container grid-list-md>
            <v-layout wrap>
            <v-flex  xs12>
              <v-text-field 
                  label="Current password"
                  v-model="current_password"
                  required
                  :type="show_password_current ? 'text' : 'password'"
                  @click:append="show_password_current = !show_password_current"
                  :append-icon="show_password_current ? 'visibility_off' : 'visibility'"
                  :rules="[() => !!current_password || 'This field is required']"
              ></v-text-field>
              <v-text-field 
                  label="New password"
                  v-model="new_password"
                  required
                  :type="show_password_new ? 'text' : 'password'"
                  @click:append="show_password_new = !show_password_new"
                  :append-icon="show_password_new ? 'visibility_off' : 'visibility'"
                  :rules="[() => !!new_password || 'This field is required']"
              ></v-text-field>
              </v-flex>
                </v-layout>
              </v-container>
          </v-card-text>

          
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" flat 
                  @click="closePassword"
                  :loading="loader">Close</v-btn>
            <v-btn color="blue darken-1" flat 
                  @click="validatePassword"
                  :loading="loader_password">Update</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

  </v-app>
</template>
 
<script>
  import Toolbar from './components/Toolbar.vue'
  import Navigation from './components/Navigation.vue'
  
  import { BackendApi } from '@/backend.js'
  import { EventBus } from './main';
  
  export default {
    components: {
      Toolbar,
      Navigation
    },
    data () {
      return {
        dialog: false,
        error_auth: false,
        msg: '',
        dialog_warning: false,
        nav: false,
        dialog_profile: false,
        userprofile: { api_secret: '', api_login: '', level: "", email: "", default_project: { name: ""} },
        loader: false,
        projects_list: [],
        loader_password: false,
        dialog_password: false,
        current_password: '',
        new_password: '',
        show_password_current: false,
        show_password_new: false
      }
    },
    created() {
        EventBus.$on('ApiAuthenticationError', obj => {
            this.dialog = true
            this.error_auth = true
            this.msg = this.sentenceCase(obj.response.data.error)            
        });
        
        EventBus.$on('ApiError', obj => {
            this.dialog = true 
            this.msg = this.sentenceCase(obj.response.data.error)
        });

        EventBus.$on('ApiFatal', obj => {
            this.dialog = true 
            this.error_auth = true
            this.msg =  obj
        });

        EventBus.$on('AppWarning', obj => {
            this.dialog_warning = true
            this.msg = obj
        });
    },
    methods: {
      sentenceCase (str) {
        return str.replace(/[a-z]/i, function (letter) {
           return letter.toUpperCase();
        }).trim();
      },
      updatePassword(){
        this.dialog_profile = false
        this.dialog_password = true

        this.current_password = ""
        this.new_password = ""
      },
      userLogout() {
        // backend call
        // todo

        // remove user from local storage to log user out
        localStorage.removeItem('user_session');

        // emit signal to others components
        EventBus.$emit('ApiLogout', 'success')

        // redirect to the login page
        this.$router.push('/login');
      },
      hideNavigation() {
         this.nav = !this.nav
      },
      closeError() {
        this.dialog = false
        this.msg = ''
        
        this.error_auth = false
      },
      closeWarning() {
        this.dialog_warning = false
        this.msg = ''
      },
      closeProfile(){
        this.dialog_profile = false
      },
      closePassword(){
        this.dialog_password = false
      },
      async userProfile() {
        // show the user profile
        this.dialog_profile = true
        this.loader = true
        this.projects_list = []
        this.userprofile = { api_secret: '', api_login: '', level: "", email: "", default_project: { name: ""} }

        // retrieve user session from local storage
        const user =  localStorage.getItem('user_session')
        const user_json =  JSON.parse(user)

        // call the backend to get the user profile
        await BackendApi.getUser(user_json.user_id).then(resp => {
          if ( resp != undefined) {
            this.userprofile = resp.user
            var i=0
            for (i = 0; i < this.userprofile.projects.length; ++i) {
                this.projects_list.push( this.userprofile.projects[i]["name"]  )
            }

          } 

          // disable loader
          this.loader = false
        })
      },
      async validatePassword () {
        // enable progress
        this.loader_password = true

        if ( !this.current_password ){
           EventBus.$emit('AppWarning', "The current password is required!"); 
           // disable progress
           this.loader_password = false
           return;  
        }

        if ( !this.new_password ){ 
            EventBus.$emit('AppWarning', "The new password is required"); 
            // disable progress
            this.loader_password = false
            return;  
        }

         // retrieve user session from local storage
        const user =  localStorage.getItem('user_session')
        const user_json =  JSON.parse(user)

        await BackendApi.updatePassword(user_json.user_id, this.current_password, this.new_password).then(resp => {
            if ( resp != undefined) {
              EventBus.$emit('AppWarning', "Password updated")
            } 
         })


        // disable progress
        this.loader_password = false
      }
   }
  }
</script>