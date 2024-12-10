<template>
	<div id="login">
		<el-container>
			<el-main>
				<div id="login-form">
					<h1 style="color: black; text-align: center;">Python在线考试系统</h1>
					<el-form ref="loginForm" status-icon :model="loginForm" :rules="rules">
						<el-form-item label="学号" prop="username">
							<el-input v-model="loginForm.username" autocomplete="off"></el-input>
						</el-form-item>
						<el-form-item label="密码" prop="password">
							<el-input type="password" v-model="loginForm.password" autocomplete="off"></el-input>
						</el-form-item>
						<slide-verification @check-result="checkResult"></slide-verification>
						<br />
						<el-button type="primary" @click.native.prevent="handLogin('loginForm')">登录</el-button>
						<div class="text-foot">
							还没有账号?
							<router-link to="/register" class="register-link">
								注册
							</router-link>
						</div>
					</el-form>
				</div>
			</el-main>
		</el-container>
	</div>
</template>

<script>
	import SlideVerification from '@/components/SlideVerification.vue';
	export default {
		data() {
			return {
				confirmSuccess: false,
				loginForm: {
					username: null,
					password: null,
				},
				rules: {
					username: [{ required: true, message: '请输入学号', trigger: 'blur' }],
					password: [{ required: true, message: '请输入密码', trigger: 'blur' }]
				}
			};
		},
		components: {
			SlideVerification
		},
		methods: {
			checkResult(message) {
				this.confirmSuccess = message;
			},
			handLogin(formName) {
				localStorage.clear();
				sessionStorage.clear();
				if (this.confirmSuccess) {
					const msg = this;
					this.$refs[formName].validate((valid) => {
						if (valid) {
							axios.post(`api/jwt-auth/`, this.loginForm).then((res) => {
								if (res.status == 200) {
									this.$message({
										message: '登录成功',
										type: 'success'
									});
									this.$store.commit("setUser", res.data.user);
									this.$store.commit("setStudent", res.data.student);
									this.$store.commit("setAuthorization", res.data.token);
									this.$router.push('/exam');
								}
							}).catch(function (error) {
								msg.$message('登录失败，账号或密码错误');
							});
						}
					});
				} else {
					this.$message('请拖动滑块进行验证！');
				}
			}
		}
	};
</script>

<style lang="scss" scoped>
#login {
	height: 100vh;
	background-image: url(../assets/bg.jpg);
	background-repeat: no-repeat;
	background-size: cover;
	display: flex;
	justify-content: center;
	align-items: center;
}

#login-form {
	width: 90%;
	max-width: 400px;
	padding: 20px;
	border-radius: 10px;
	background: rgba(255, 255, 255, 0.2);
	backdrop-filter: blur(15px);
	box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
	margin: auto;
}

.el-form-item label {
	color: #333;
	font-weight: bold;
}

.el-button {
	width: 100%;
	margin-top: 20px;
}

.text-foot {
	text-align: center;
	margin-top: 20px;
	font-weight: 700;
}

.register-link {
	color: #409eff;
	text-decoration: none;

	&:hover {
		text-decoration: underline;
	}
}

@media (max-width: 600px) {
	#login-form {
		width: 90%;
		padding: 15px;
	}
}
</style>
