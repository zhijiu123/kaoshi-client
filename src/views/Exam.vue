<template>
	<div id="exam">
		<el-row type="flex" justify="center">
			<el-col :span="4">
				<el-input v-model="key" placeholder="请输入考试名称" prefix-icon="el-icon-search" clearable></el-input>
			</el-col>
			<el-col :span="4">
				<el-button type="primary" @click="searchExam()">搜索试卷</el-button>
			</el-col>
		</el-row>
		</el-row>
		<el-row>
			<h3 style="border-left: solid 10px rgb(220, 208, 65);">考试列表</h3>
			<div style="padding-left: 15px">
				<el-col :span="4" v-for="(item, index) in pagination.results" :key="index" :offset="index > 0 ? 1 : 0">
					<el-card :body-style="{ padding: '0px' }" v-loading="loading">
						<img src="@/assets/exam.png" class="image">
						<div style="padding: 14px;">
							<span>{{item.name}}</span>
							<p>
								<span>考试时间：{{item.exam_date}}</span>
								<br />
								<span>考试时长：{{item.total_time}}分钟</span>
							</p>
							<div class="bottom clearfix">
								<el-button type="text" class="button" @click="toAnswer(index)">开始考试</el-button>
							</div>
						</div>
					</el-card>
				</el-col>
			</div>
		</el-row>
		<Pagination :count="pagination.count" @size-change="handleSizeChange" @current-change="handleCurrentChange"></Pagination>
	</div>
</template>

<script>
	import Pagination from '@/components/Pagination.vue'
	import {MessageBox} from 'element-ui'
	export default {
		data() {
			return {
				loading: false,
				key: null,
				page: 1,
				page_size: 5,
				pagination: {
					count: null,
					next: null,
					previous: null,
					results: []
				}
			}
		},
		components: {
			Pagination
		},
		methods: {
			//获取考试信息
			getExamInfo() {
				this.$axios(`/api/exams/?format=json`, {
					params: {
						page: this.page,
						page_size: this.page_size,
						student_id: this.$store.state.student.id,
					}
				}).then(res => {
					this.pagination = res.data
					this.loading = false
			
				
					// console.log(this.pagination)
				}).catch(error => {
					console.log(error)
				})
			},
			//改变每页条数
			handleSizeChange(val) {
				// console.log(`每页 ${val} 条`);
				this.page_size = val
				this.searchExam()
			},
			//跳转到多少页
			handleCurrentChange(val) {
				// console.log(`当前页: ${val}`);
				this.page = val
				this.searchExam()
			},
			//搜索考试
			searchExam() {
				if (this.key) {
					this.$axios(`/api/exams/?format=json`, {
						params: {
							page: this.page,
							page_size: this.page_size,
							search: this.key,
							student_id: this.$store.state.student.id,
						}
					}).then(res => {
						if (res.status == 200) {
							this.pagination = res.data
							
						}
					})
				} else {
					this.getExamInfo()
				}
			},
		//跳转到答题页
		toAnswer(index) {
			//用localStorage存储考试信息和试卷信息

			this.$axios(`/api/grade-records/?fomat=json`, {
				params: {
					exam_id: this.pagination.results[index].paper.id,
					student_id: this.$store.state.student.id,
				}
			}).then(res => {
				this.tempPagination = res.data
				if (this.tempPagination.results.length == 0) {
					this.$message({
						type: 'message',
						message: '您未参加过该考试，即将进入考试页面！'
					});
					localStorage.removeItem('exam');
					localStorage.removeItem('paper');
					sessionStorage.removeItem('isPractice')
					localStorage.setItem("exam", JSON.stringify(this.pagination.results[index]));
					localStorage.setItem("paper", JSON.stringify(this.pagination.results[index].paper));
					this.$store.commit("setIsPractice", false)
					this.$router.push({
						path: '/answer',
						query: {}
					})
				} else {
				
					MessageBox.alert(`您已参加过考试，不能重复考试！您上次的考试成绩为: ${this.tempPagination.results[0].score} 分`,'提示',{
						confirmButtonText: '确定',
						callback: action => {
							console.log('用户点击确定');
						}
					})
					
				}

			}).catch(error => {
				console.log(error)
			})


		}
	},
		created() {
			this.getExamInfo()
			this.loading = true
		}
	}
</script>
<style lang="scss" scoped>
#exam {
  padding: 20px;
  text-align: center;

  .el-row {
    margin-bottom: 20px;

    &:last-child {
      margin-bottom: 0;
    }
  }

  h3 {
    color: #333;
    font-size: 24px;
    margin-bottom: 20px;
    padding-left: 15px;
    border-left: solid 10px #dcc841; // 调整边框颜色
  }

  .el-input {
    margin-right: 10px;
  }

  .el-card {
	width: 200px;
    margin-bottom: 20px;
    border-radius: 10px; // 添加圆角边框
    box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1); // 添加阴影效果

    .image {
      width: 100%;
      height: auto;
      max-height: 150px;
      margin: 0 auto;
      display: block;
      border-top-left-radius: 10px;
      border-top-right-radius: 10px;
    }

    .bottom {
      text-align: right;
      margin-top: 10px;
    }

    .button {
      padding: 0;
    }
  }

  .clearfix:before,
  .clearfix:after {
    display: table;
    content: "";
  }

  .clearfix:after {
    clear: both;
  }
}
</style>
