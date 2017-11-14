<template>
  <section class="container">
    <div v-if="login">
      <h4>
        Hello, {{name}}
      </h4><br>
      <input class="search" type="text" placeholder="Find your group..." v-model="groupSearch">
      <ul>
        <div v-for="(item, index) in groupResult" :key="index">
          <div class="result" @click="groupSelected(item.id)">[{{item.privacy}}] {{ item.name }}
          </div>
          <div class="border" v-if="item.id === groupID">
            <div v-for="(item2, index2) in feed" :key="index2">
              <div class="feed" @click="postSelected(item2.id)">
                {{ item2.message }}
                <div v-if="item2.picture">
                  <img class="picture" :src="item2.picture" alt="picture">
                </div>
              </div>
              <div class="border2" v-if="item2.id === postID">
                <div v-for="(item3, index3) in comment" :key="index3">
                  <div class="comment">
                    {{ item3.message }}
                  </div>
                  <div class="reply" v-for="(item4, index4) in item3.reply" :key="index4">
                    > {{item4.message}}
                  </div>
                  <div class="commentline"></div>
                </div>
              </div>
              <div class="feedline"></div>
            </div>
          </div>
        </div>
      </ul>
    </div>
    <div v-else>
      <button @click="fbLogin">Login with Facebook</button>
    </div>
  </section>
</template>

<script>
  import Logo from '~/components/Logo.vue'

  export default {
  	components: {
  		Logo
  	},

  	data() {
  		return {
  			APPID: '891066087729524',
  			APPSECRET: 'c380a169180c54f28e6988cd44b26435',
  			login: false,
  			accessToken: null,
  			name: '',
  			groupSearch: '',
  			groupResult: [],
  			groupID: null,
  			feed: [],
  			postID: null,
  			comment: []
  		}
  	},
  	methods: {
  		fbLogin() {
  			FB.login()
  		},
  		fbLogout() {
  			FB.logout()
  		},
  		groupSelected(id) {
  			this.groupID = id

  			FB.api(`/${id}/feed`, { accessToken: this.accessToken }, response => {
  				console.log('response', response)
  				this.feed = response.data

  				this.feed.forEach((post, index) => {
  					FB.api(`/${post.id}`, { fields: 'picture' }, response => {
  						if (response.picture) {
  							this.feed[index].picture = response.picture
  							this.$forceUpdate()
  						}
  					})
  				})
  			})
  		},
  		postSelected(id) {
  			this.postID = id

  			FB.api(`/${id}/comments`, {}, response => {
  				this.comment = response.data

  				this.comment.forEach((comment, index) => {
  					FB.api(`/${comment.id}/comments`, {}, response => {
  						this.comment[index].reply = response.data
  						this.$forceUpdate()
  					})
  				})
  			})
  		}
  	},
  	watch: {
  		groupSearch: function(search) {
  			FB.api('/search', { type: 'group', q: search }, response => {
  				this.groupResult = response.data
  				this.$forceUpdate()
  			})
  		}
  	},
  	mounted() {
  		var _this = this

  		window.fbAsyncInit = function() {
  			FB.init({
  				appId: _this.APPID,
  				autoLogAppEvents: true,
  				xfbml: true,
  				version: 'v2.11'
  			})

  			FB.getLoginStatus(function(response) {
  				if (response.status === 'connected') {
  					_this.login = true
  					_this.accessToken = response.authResponse.accessToken

  					FB.api('/me', function(response) {
  						_this.name = response.name
  					})

  					FB.api(`/${response.authResponse.userID}/permissions`, function(
  						response
  					) {
  						console.log('response', response)
  					})

  					// GET https://graph.accountkit.com/v1.2/access_token?grant_type=authorization_code&code=<authorization_code>&access_token=AA|<facebook_app_id>|<app_secret>
  					// FB.api(
  					// 	`/access_token`,
  					// 	{
  					// 		grant_type: 'authorization_code',
  					// 		code: _this.accessToken,
  					// 		access_token: 'AA'
  					// 	},
  					// 	function(response) {
  					// 		console.log(response)
  					// 	}
  					// )

  					// FB.api(
  					// 	`/oauth/access_token`,
  					// 	{
  					// 		client_id: _this.APPID,
  					// 		client_secret: _this.APPSECRET,
  					// 		redirect_uri: 'http://localhost:3000/'
  					// 	},
  					// 	function(response) {
  					// 		console.log(response)
  					// 	}
  					// )
  				} else {
  					_this.login = false
  				}
  			})
  		}
  		;(function(d, s, id) {
  			var js,
  				fjs = d.getElementsByTagName(s)[0]
  			if (d.getElementById(id)) {
  				return
  			}
  			js = d.createElement(s)
  			js.id = id
  			js.src = 'https://connect.facebook.net/en_US/sdk.js'
  			fjs.parentNode.insertBefore(js, fjs)
  		})(document, 'script', 'facebook-jssdk')
  	}
  }
</script>

<style>
  .container {
  	min-height: 100vh;
  	display: flex;
  	justify-content: center;
  	align-items: center;
  	text-align: center;
  }

  .title {
  	font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
  		'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif; /* 1 */
  	display: block;
  	font-weight: 300;
  	font-size: 100px;
  	color: #35495e;
  	letter-spacing: 1px;
  }

  .subtitle {
  	font-weight: 300;
  	font-size: 42px;
  	color: #526488;
  	word-spacing: 5px;
  	padding-bottom: 15px;
  }

  .links {
  	padding-top: 15px;
  }

  .search {
  	margin-bottom: 20px;
  }

  .result {
  	color: blue;
  	cursor: pointer;
  }

  .result:hover {
  	color: purple;
  }

  .border {
  	border: 1px solid blue;
  	margin: 10px;
  	padding: 5px;
  }

  .feed {
  	color: green;
  	padding: 15px;
  	cursor: pointer;
  }

  .feed:hover {
  	color: purple;
  }

  .feedline {
  	border-bottom: 1px solid blue;
  }

  .picture {
  	width: 20%;
  	margin-top: 20px;
  }

  .border2 {
  	border: 1px solid green;
  	margin: 10px;
  	padding: 5px;
  }

  .comment {
  	color: red;
  	padding: 15px;
  }

  .commentline {
  	border-bottom: 1px solid green;
  }

  .reply {
  	color: pink;
  	padding: 5px;
  }
</style>
