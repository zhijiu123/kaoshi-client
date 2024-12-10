<template>
	<div id="layout">
		<el-container>
			<el-header>
				<el-row type="flex" justify="space-between">
					<a href="/"><img src="../assets/logo.png" height="50px" /></a>
					<el-menu :default-active="activeIndex" class="el-menu-title" mode="horizontal" @select="handleSelect"
					 background-color="#ffffff" text-color="#85baef" active-text-color="#1884f2" :router="true">
						<el-menu-item index="/exam">考试中心</el-menu-item>
						<el-menu-item index="/practice">模拟练习</el-menu-item>
						<el-menu-item index="/grade">查询成绩</el-menu-item>
						<el-menu-item index="/mistake" disabled>错题本</a></el-menu-item>
					</el-menu>
					<el-dropdown>
						<span class="el-dropdown-link" style="height: 50px;">
							<el-row>
								<!-- <img src="../assets/avatar.png" height="35px" /> -->
								<span>
									<i class="el-icon-user-solid"></i>
									<span>{{getStudent.name}}</span>
									<i class="el-icon-arrow-down el-icon--right"></i>
								</span>
							</el-row>
						</span>
						<el-dropdown-menu slot="dropdown">
							<el-dropdown-item>
								<el-button type="text" @click="toCenter">个人中心</el-button>
							</el-dropdown-item>
							<el-dropdown-item>
								<el-button type="text" @click="toUpdatePwd">修改密码</el-button>
							</el-dropdown-item>
							<el-dropdown-item divided>
								<el-button type="text" @click="loginOut">退出登录</el-button>
							</el-dropdown-item>
						</el-dropdown-menu>
					</el-dropdown>
				</el-row>
			</el-header>
			<el-main>
				<router-view />
			</el-main>
			<el-footer>
				<b>@Copyright 2024-Li qiao</b>
			</el-footer>
		</el-container>
	</div>
</template>

<script>
	export default {
		name: "layout",
		data() {
			return {
				activeIndex: this.$route.path
			};
		},
		computed: {
			getStudent() {
				return this.$store.state.student;
			}
		},
		methods: {
			handleSelect(key, keyPath) {
				//console.log(key, keyPath);
				this.activeIndex = key
			},
			loginOut() {
				//console.log("before:" + localStorage.getItem('Authorization'))
				localStorage.clear()
				sessionStorage.clear()
				//console.log("after" + localStorage.getItem('Authorization'))
				this.$router.push('/login');
			},
			toCenter() {
				this.$router.push({
					name: 'Center',
					params: {}
				});
			},
			toUpdatePwd() {
				this.$router.push({
					name: 'Password',
					params: {}
				});
			}
		},
		created() {

		}
	}
</script>


<style lang="scss" scoped>
#layout {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  margin: 0 auto;
  max-width: 1200px;
  background-color: #f7f7f7;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.el-header {
  background-color: #fff;
//   padding: 10px 40px;
  border-bottom: 1px solid #e6e6e6;
  //display: flex;
  justify-content: space-between;
  align-items: center;
}

.logo {
  height: 50px;
}

.el-menu {
  background-color: transparent;
  border-bottom: none;
  &-title {
    display: flex;
    align-items: center;
    .el-menu-item {
      font-size: 16px;
      font-weight: 500;
      &.is-active {
        border-bottom: 2px solid #409eff;
      }
      &:hover {
        background-color: #ecf5ff;
      }
    }
  }
}

.el-dropdown {
  .el-dropdown-link {
    display: flex;
    align-items: center;
    color: #606266;
    font-size: 16px;
    cursor: pointer;
    .el-icon-user-solid {
      margin-right: 5px;
    }
  }
}

.el-main {
  flex: 1;
  padding: 20px 40px;
  background-color: #fff;
}

.el-footer {
  background-color: #fff;
  padding: 20px 40px;
  border-top: 1px solid #e6e6e6;
  color: #909399;
  text-align: center;
  font-size: 14px;
}

// Responsive adjustments
@media (max-width: 768px) {
  #layout {
    padding: 0 20px;
  }

  .el-header,
  .el-footer {
    padding: 10px 20px;
  }

  .el-menu {
    .el-menu-item {
      font-size: 14px;
    }
  }
}
</style>