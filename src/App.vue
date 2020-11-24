<template>
  <div id="app">
    <!-- NAVIGATION -->
    <nav class="navbar navbar-dark bg-primary">
      <a class="navbar-brand" href="#">VuejsFirebase</a>
    </nav>
    <!-- MAIN CONTENT -->
    <div class="container">
      <div class="row mt-5">
        <div class="col-sm-4">
          <div class="card">
            <div class="card-header">
              <h3>Add a New Website</h3>
            </div>
            <div class="card-body">
              <form @submit.prevent="addWebsite">
                <div class="form-group">
                  <input type="text" id="Name" class="form-control" v-model="newWebsite.name" placeholder="Name">
                </div>
                <div class="form-group">
                  <input type="text" id="Author" class="form-control" v-model="newWebsite.author" placeholder="Author">
                </div>
                <div class="form-group">
                  <input type="text" id="URL" class="form-control" v-model="newWebsite.url" placeholder="URL">
                </div>
                <button type="submit" class="btn btn-primary">
                  Save
                </button>
              </form>
            </div>
          </div>
        </div>
        <div class="col-sm-8 text-center">
                    <img src="./assets/logo.png" alt="">
          <div class="card">
            <div class="card-header">
              <h3>Websites List</h3>
            </div>
            <div class="card-body">
              <table class="table table-striped table-bordered">
                <thead>
                  <tr>
                    <th>Name</th>
                    <th>Author</th>
                    <th>Operations</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="w in websites" :key="w.key">
                    <td>
                      <a v-bind:href="w.url">{{w.name}}</a>
                    </td>
                    <td>
                      {{w.author}}
                    </td>
                    <td>
                      <button @click="deleteWebsite(w)" class="btn btn-danger">
                        Delete
                      </button>
                      <button @click="editWebsite(w)" class="btn btn-primary">
                        Edit
                      </button>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
// Firebase
import Firebase from 'firebase';
import config from './config';
var add_or_edit = new Boolean(1); //0 = edit ; 1 = add
let app = Firebase.initializeApp(config);
let db = app.firestore();
let websitesRef = db.collection('websites');
const { shell } = require('electron');
document.addEventListener('click',function(event){
  if (event.target.tagName === 'A' && event.target.href.startsWith('http')){
    event.preventDefault()
    shell.openExternal(event.target.href)
  }
})
// toast
import toastr from 'toastr';

export default {
  data () {
    return {
      websites: [],
      newWebsite: {
        name: '',
        author: '',
        url: ''
      }
    }
  },
  created() {
            websitesRef.onSnapshot((snapshotChange) => {
                this.websites = [];
                snapshotChange.forEach((doc) => {
                    console.log(doc.data().name);
                    this.websites.push({
                        id: doc.id,
                        name: doc.data().name,
                        url: doc.data().url,
                        author: doc.data().author
                    })
                });
            })
        },
  methods: {
    addWebsite() {
      if (add_or_edit == true){
        websitesRef.add(this.newWebsite);
      }else{
        websitesRef.doc(this.newWebsite.id).update(this.newWebsite);
        toastr.success('Website updated!');
      }
      this.newWebsite.name = '';
      this.newWebsite.author = '';
      this.newWebsite.url = '';
      add_or_edit=new Boolean(1);
      },
    deleteWebsite(website) {
      if(confirm('Are you sure you want to delete it?')) {
        websitesRef.doc(website.id).delete();
        toastr.success('Website removed');
      }
    },
    editWebsite(website) {
      add_or_edit=new Boolean(0);
      document.getElementById('Name').value = website.name;
      document.getElementById('Author').value = website.author;
      document.getElementById('URL').value = website.url;
      this.newWebsite = website;
    }
  }
}
</script>

<style>
#app {
  background: #485563;  /* fallback for old browsers */
  background: -webkit-linear-gradient(to right, #29323c, #485563);  /* Chrome 10-25, Safari 5.1-6 */
  background: linear-gradient(to right, #29323c, #485563); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */

height: 100vh;
}
</style>
