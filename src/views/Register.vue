<template>
	<div id="register">
		<el-container>
			<el-main>
				<div id="register-from">
					<el-form ref="registerForm" status-icon :model="registerForm" :rules="rules">
						<h1 style="color: black; text-align: center;">Python在线考试系统</h1>
						<el-cascader v-model="cascaderCheckValue" placeholder="请选择学院、专业、班级" :options="cascaderOptions" filterable></el-cascader>
						<el-form-item label="学号" prop="username">
							<el-input v-model="registerForm.username" autocomplete="off"></el-input>
						</el-form-item>
						<el-form-item label="姓名" prop="name">
							<el-input v-model="registerForm.name" autocomplete="off"></el-input>
						</el-form-item>
						<el-form-item label="密码" prop="password">
							<el-input type="password" v-model="registerForm.password" autocomplete="off"></el-input>
						</el-form-item>
						<el-form-item label="确认密码" prop="checkpwd">
							<el-input type="password" v-model="registerForm.checkpwd" autocomplete="off"></el-input>
						</el-form-item>
						<slide-verification @check-result="checkResult"></slide-verification>
						<br />
						<el-button type="primary" @click.native.prevent="handRegister('registerForm')">注册</el-button>
						<div class="text-foot">
							已有账号?
							<router-link to="/login" class="login-link">登录</router-link>
						</div>
					</el-form>
				</div>
			</el-main>
		</el-container>
	</div>
</template>

<script>
import SlideVerification from '@/components/SlideVerification.vue';
import {MessageBox} from 'element-ui'
export default {
	data() {
		var validatePass = (rule, value, callback) => {
			if (value === '') {
				callback(new Error('请输入密码'));
			} else {
				if (this.registerForm.checkpwd !== '') {
					this.$refs.registerForm.validateField('checkpwd');
				}
				callback();
			}
		};
		var validatePass2 = (rule, value, callback) => {
			if (value === '') {
				callback(new Error('请再次输入密码'));
			} else if (value !== this.registerForm.password) {
				callback(new Error('两次输入密码不一致!'));
			} else {
				callback();
			}
		};
		return {
			confirmSuccess: false,
			registerForm: {
				username: null,
				password: null,
				checkpwd: null,
				name: null
			},
			cascaderOptions:[],
			cascaderCheckValue:[],
			rules: {
				username: [{
					required: true,
					message: '请输入学号',
					trigger: 'blur'
				}, {
					min: 6,
					max: 15,
					message: '长度在 6 到 15 个字符',
					trigger: 'blur'
				}],
				password: [{
					required: true,
					message: '请输入密码',
					trigger: 'blur'
				}, {
					min: 6,
					max: 15,
					message: '长度在 6 到 15 个字符',
					trigger: 'blur'
				}, {
					validator: validatePass,
					trigger: 'blur'
				}],
				checkpwd: [{
					required: true,
					message: '请再次输入密码',
					trigger: 'blur'
				}, {
					min: 6,
					max: 15,
					message: '长度在 6 到 15 个字符',
					trigger: 'blur'
				}, {
					validator: validatePass2,
					trigger: 'blur'
				}],
				name: [{
					required: true,
					message: '请输入姓名',
					trigger: 'blur'
				}, {
					min: 2,
					max: 10,
					message: '长度在 2 到 8 个字符',
					trigger: 'blur'
				}]
			}
		}
	},
	created() {
			this.getColleges(),
			this.loading = true
		},
	components: {
		SlideVerification
	},
	methods: {
		//获取学院、专业、班级信息
		getColleges() {
			axios.get('api/college-major-clazzs/').then(res => {
				console.log(res.data);
				this.cascaderOptions = res.data.options;
			}).catch(function(error) {
				console.log(error);
			});
		},

		checkResult(message) {
			this.confirmSuccess = message
		},
		handRegister(formName) {
			if (this.confirmSuccess) {
				const msg = this
				this.$refs[formName].validate((valide) => {
					if (valide) {
				
						//console.log(this.cascaderCheckValue);
						axios.post(`api/register/?format=json`, {
							params:{
								registerForm:this.registerForm,
								college_id: this.cascaderCheckValue[0],
								major_id: this.cascaderCheckValue[1],
								clazz_id: this.cascaderCheckValue[2]
							}
						}).then(res => {
							console.log(res);
							if (res.status == 200) {
								this.$message({
									message: '注册成功',
									type: 'success'
								});
								this.$router.push('/login')
							} else {
								this.$message({
									message: res.data.msg,
									type: 'error'
								});
							}
						}).catch(error =>{
							this.$message(
								{
									message: '注册失败：' + error.response.data.msg,
									type: 'error'
								}
	
							);
						}
					
					);
					} else {
						//表单验证失败
					}
				});
			} else {
				this.$message('请拖动滑块进行验证！');
			}
		}
	}
}
</script>

<style lang="scss" scoped>
#register {
	background-image: url(~@/assets/bg.jpg);
	background-repeat: no-repeat;
	background-size: cover;
	height: 100vh;
	display: flex;
	justify-content: center;
	align-items: center;
}

#register-from {
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
	color: black;
	font-size: 20px;
	font-weight: bold; /* 加粗标签 */
}
.el-cascader {
	width: 100%;
}
.el-input {
	color: black;
}

.el-form-item {
	text-shadow: none;
}

.el-button {
	width: 100%;
}

.text-foot {
	font-weight: 700;
	margin-top: 20px;
}

@media (max-width: 600px) {
	#register-from {
		width: 95%;
	}
}
</style>
