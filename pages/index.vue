<template>
  <section class="container">
    <div v-if="login">
      <h4>
        Hello, {{name}}
      </h4><br>
      <button @click="fbTest">Test API</button>
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
  			APPID: '131695504135597',
  			APPSECRET: 'e80ffea5de9c7931fcdf0c7d5aefe3be',
  			accessToken: null,
  			login: false,
  			name: ''
  		}
  	},
  	methods: {
  		fbLogin() {
  			FB.login()
  		},
  		fbTest() {
  			FB.api('/me', { fields: 'id, name, picture' }, response => {
  				console.log(response)
  			})

  			FB.api(
  				`/v2.11/${this.APPID}/subscriptions`,
  				'post',
  				{
  					object: 'user',
  					callback_url: 'https://hidden-spire-18590.herokuapp.com/callback/',
  					fields: 'contact_email',
  					verify_token: 'facebooktestwebhook',
  					access_token: this.accessToken
  				},
  				response => {
  					console.log(response)
  				}
  			)

  			console.log('accessToken', this.accessToken)
  		},
  		fbLogout() {
  			FB.logout()
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

  					console.log('_this.APPID', _this.APPID)
  					console.log('_this.APPSECRET', _this.APPSECRET)
  					console.log('_this.accessToken', _this.accessToken)

  					FB.api(
  						`/oauth/access_token`,
  						{
  							client_id: _this.APPID,
  							client_secret: _this.APPSECRET,
  							grant_type: 'client_credentials'
  						},
  						response => {
  							console.log(response)
  						}
  					)
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
</style>
