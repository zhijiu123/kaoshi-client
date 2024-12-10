<template>
	<div id="practice">

		<el-card class="box-card">
			<div slot="header" class="clearfix">
				<span>组卷模块</span>
			</div>
			<el-form ref="form" :model="form" label-width="80px">
				<el-select v-model="value" filterable placeholder="请选择课程">
					<el-option v-for="item in options" :key="item.id" :label="item.course" :value="item.id">
					</el-option>
				</el-select>
				<el-form-item label="单选题:">
					<el-slider v-model="form.choice_number" :step="1" :max="20" show-stops></el-slider>
				</el-form-item>
				<el-form-item label="多选题:">
					<el-slider v-model="form.program_number" :step="1" :max="20" show-stops></el-slider>
				</el-form-item>
				<el-form-item label="填空题:">
					<el-slider v-model="form.fill_number" :step="1" :max="20" show-stops></el-slider>
				</el-form-item>
				<el-form-item label="判断题:">
					<el-slider v-model="form.judge_number" :step="1" :max="20" show-stops></el-slider>
				</el-form-item>
				<el-form-item label="难易度:">
					<el-rate v-model="form.level" :texts="level" show-text></el-rate>
				</el-form-item>
			</el-form>
			<el-button type="primary" @click="toAnswer">生成试卷并开始答题</el-button>
		</el-card>
		<el-table :data="pagination.results" style="width: 61%" v-loading="loading" border>
			<el-table-column type="index" width="50">
			</el-table-column>
			<el-table-column prop="creat_time" label="练习时间" width="250px" sortable>
				<template slot-scope="scope">
					<i class="el-icon-time"></i>
					<span style="margin-left: 10px">{{ scope.row.create_time.replace('T', ' ').substring(0, 19)
						}}</span>
				</template>
			</el-table-column>
			<el-table-column prop="name" label="练习名称" width="250px">
			</el-table-column>
			<el-table-column>
				<template slot-scope="scope">
					<!-- <router-link target="_blank" :to="{path:'/record',query:{practice_id:scope.row.id}}"> -->
					<el-button type="text" @click="toRecord(scope.row.id)" size="small">查看记录</el-button>
					<!-- </router-link> -->
				</template>
			</el-table-column>
		</el-table>
		<Pagination :count="pagination.count" @size-change="handleSizeChange" @current-change="handleCurrentChange">
		</Pagination>
	</div>
</template>

<script>
	import Pagination from '@/components/Pagination.vue'
	import {MessageBox} from 'element-ui'
	export default {
		data() {
			return {
				form: {
					choice_number: 1,
					fill_number: 1,
					judge_number: 1,
					program_number: 1,
					level: 1,
					course: null,
				},
				value:'',
				exam: {
					name: '模拟测试',
					total_time: 120
				},
				practice: {},
				level: ["入门", "简单", "普通", "较难", "困难"],
				loading: false,
				page: 1,
				page_size: 5,
				pagination: {
					count: null,
					next: null,
					previous: null,
					results: []
				},
				options:[]
			}
		},
		components: {
			Pagination
		},
		created() {
			this.getCourse()	
			this.getPracticeInfo()
			this.loading = true
		},
		methods: {
			toRecord(practiceId){
				try{
					this.$router.push({
						path:'/record',
						query:{
							practice_id:practiceId
						}
					})
				}catch(errorMsg){
					console.error('跳转失败',error);
					
				}
				
			},
			toAnswer() {
				if (!this.value) {
					MessageBox.alert(`请选择课程再开始练习`,'提示',{
						confirmButtonText: '确定',
						callback: action => {
							console.log('用户点击确定');
						}
					})
					return;
				}
				this.form.course = this.value
				//用localStorage存储考试信息和试卷信息
				axios.post(`api/practices/`, {
					name: '模拟练习',
					student_id: this.$store.state.student.id,
				}).then(res => {
					console.log(res.data); //处理成功的函数 相当于success
					this.practice = res.data;
					localStorage.removeItem('practice');
					localStorage.removeItem('paper');
					sessionStorage.removeItem("isPractice")
					localStorage.setItem("practice", JSON.stringify(this.practice));
					localStorage.setItem("paper", JSON.stringify(this.form));
					this.$store.commit("setIsPractice", true)
					this.$router.push({
						name: 'Answer',
						params: {}
					})
				}).catch(function(error) {
					console.log(error) //错误处理 相当于error
				});
			},
			//获取课程
			getCourse() {
				this.$axios.get(`api/courses/?major_id=${this.$store.state.student.major}`).then(res => {
					console.log('获取数据成功');
					 this.options = res.data.map((item) => ({
						id:item.id,
						course:item.course
					}));
				}).catch(error => {
					console.error(error);
				})
			},
			//获取模拟练习
			getPracticeInfo() {
				this.$axios(`api/practices/?format=json`, {
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
				this.getPracticeInfo()
			},
			//跳转到多少页
			handleCurrentChange(val) {
				// console.log(`当前页: ${val}`);
				this.page = val
				this.getPracticeInfo()
			}
		},
	}
</script>

<style lang="scss" scoped>
#practice {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 20px;

  .el-card {
    width: 80%;
    max-width: 700px; // 限制最大宽度，确保在较大屏幕上不会过于伸展
    margin: 20px auto;
    box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
    border-radius: 4px;

    .clearfix {
      display: flex;
      justify-content: space-between;
      align-items: center;

      span {
        font-size: 1.2rem;
        font-weight: bold;
      }
    }

    .el-form {
      margin: 20px;

      .el-select {
        width: 100%;
      }

      .el-form-item {
        margin-bottom: 22px;

        .el-slider {
          width: 80%;
          margin: 0 auto;
        }
      }

      .el-rate {
        margin: 10px 0;
      }
    }

    .el-button {
      width: 100%;
      margin-top: 20px;
    }
  }

  .el-table {
    width: 80%;
    max-width: 700px; // 限制最大宽度
    margin: 20px auto;
    border-radius: 4px;

    .el-table-column {
      text-align: center;
    }
  }

  .pagination-container {
    display: flex;
    justify-content: center;
    margin: 20px 0;
  }
}
</style>
