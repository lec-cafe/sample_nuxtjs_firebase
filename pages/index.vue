<template>
  <section class="container">

    <div class="mb-3">
      <div v-if="!user">
        <p>
          コメントを投稿するには、ユーザ名を入力してください。
        </p>
        <a class="btn btn-light btn-block" tabindex="" @click="login">ログイン</a>
      </div>

      <form v-if="user">
        <div class="form-group">
          <label>Name</label>
          <div class="input-group mb-2">
            <div class="input-group-prepend">
              <div class="input-group-text">@</div>
            </div>
            <input type="text" class="form-control" v-model="user.name" disabled>
          </div>
        </div>
        <div class="form-group">
          <label>Comment</label>
          <textarea class="form-control" v-model="form.comment" rows="3"/>
        </div>
        <div class="form-group">
          <a class="btn btn-light btn-block" tabindex="" @click="submitPost">投稿</a>
        </div>
      </form>
    </div>

    <div class="list-group list-group-flush">
      <div class="list-group-item list-group-item-action flex-column align-items-start" v-for="(post,index) in posts" :key="index">
        <div class="d-flex w-100 justify-content-between">
          <h5 class="mb-1">{{ post.user }}</h5>
          <small>{{ post.date }}</small>
        </div>
        <p class="mb-1">{{ post.comment }}</p>
      </div>
    </div>
  </section>
</template>

<script>
  import firebase from '~/service/firebase'

  const db = firebase.firestore();

  export default {
    data () {
      return {
        form: {
          comment: ""
        },
        posts: null
      }
    },
    computed: {
      user () {
        return this.$store.state.user
      },
    },
    async mounted(){
      firebase.auth().onAuthStateChanged(user => {
        if(user){
          this.$store.dispatch("loginWithUserName", user.displayName)
        }
      })
      this.load()
    },
    methods: {
      async load(){
        const snapshot = await db.collection('posts')
          .orderBy('date', 'desc')
          .get()
        console.log(snapshot)
        if(snapshot.empty){
          this.posts = []
        }else{
          this.posts = snapshot.docs.map((doc)=>{
            return doc.data()
          })
        }
      },
      async login () {
        const provider = new firebase.auth.GithubAuthProvider()
        const result = await firebase.auth().signInWithPopup(provider)
        // var token = result.credential.accessToken
        var user = result.user
        this.$store.dispatch("loginWithUserName", user.displayName)
      },
      submitPost() {
        if (this.form.comment === "") {
          return false
        }
        const date = new Date()
        db.collection('posts').add({
          comment: this.form.comment,
          user: this.user.name,
          date: `${date.getMonth()+1}/${date.getDate()} ${date.getHours()}:${date.getMinutes()}`
        })
        this.load()
        this.form.comment = ""
      }
    }
  }
</script>

<style>
</style>
