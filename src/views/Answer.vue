<template>
	<div id="answer">
		<el-container>
			<el-aside width="300px">
				<el-card class="box-card" shadow="hover">
					<time-count-down :totalTime="getExam.total_time" @hand-in="handIn"></time-count-down>
					<el-divider content-position="center">选择题</el-divider>
					<el-row class="question-tag" type="flex" justify=" space-around ">
						<el-col :span="24">
							<el-tag v-for="(item, index) in choices" :key="item.id"
								:type="index + 1 == current ? 'success' : ''"
								:effect="index + 1 == current || answer[0][index] != null ? 'dark' : 'plain'"
								@click="changeQuestion(index + 1)">
								{{ index + 1 }}
							</el-tag>
						</el-col>
					</el-row>
					<el-divider content-position="center">填空题</el-divider>
					<el-row class="question-tag">
						<el-col :span="24">
							<el-tag v-for="(item, index) in fills" :key="item.id"
								:type="index + fillPos == current ? 'success' : ''"
								:effect="index + fillPos == current || answer[1][index] != null ? 'dark' : 'plain'"
								@click="changeQuestion(index + fillPos)">
								{{ index + fillPos }}
							</el-tag>
						</el-col>
					</el-row>
					<el-divider content-position="center">判断题</el-divider>
					<el-row class="question-tag">
						<el-col :span="24">
							<el-tag v-for="(item, index) in judges" :key="item.id"
								:type="index + judgePos == current ? 'success' : ''"
								:effect="index + judgePos == current || answer[2][index] != null ? 'dark' : 'plain'"
								@click="changeQuestion(index + judgePos)">
								{{ index + judgePos }}
							</el-tag>
						</el-col>

					</el-row>
					<el-divider content-position="center">多选题</el-divider>
					<el-row class="question-tag">
						<el-col :span="24">		
							<el-tag v-for="(item, index) in answer_list" :key="item.id"
								:type="index + programPos == current ? 'success' : ''"
								:effect="index + programPos == current || isChecked(index) ? 'dark' : 'plain'"
								@click="changeQuestion(index + programPos)">
								{{ index + programPos }}
							</el-tag>
						</el-col>
					</el-row>
					<el-divider></el-divider>
					<div class="bottom clearfix">
						<el-tag type="primary" effect="dark"></el-tag>已答
						<el-tag type="primary" effect="plain"></el-tag>未答
						<el-tag type="success" effect="dark"></el-tag>当前
						<br />
						<br />
						<el-button type="primary" class="button" @click="handIn">交卷</el-button>
					</div>
				</el-card>
			</el-aside>
			<el-container>
				<el-header>
					<el-card class="box-card" shadow="hover" style="width: auto;">
						<div id="information">
							<el-row type="flex" justify="center">
								<el-col :span="4">
									<span>学号：{{ getUser.username }}</span>
								</el-col>
								<el-col :span="4">
									<span>姓名：{{ getStudent.name }}</span>
								</el-col>
								<el-col :span="4">
									<span>性别：{{ getStudent.gender = 'M' ? '男' : '女' }}</span>
								</el-col>
								<el-col :span="4">
									<span>专业：{{ majorName }}</span>
								</el-col>
								<el-col :span="4">
									<span>年级：{{ clazz.year }}</span>
								</el-col>
								<el-col :span="4">
									<span>班级：{{ clazz.clazz }}</span>
								</el-col>
							</el-row>
						</div>
					</el-card>
				</el-header>
				<el-main>
					<el-card class="box-card" shadow="hover" style="width: 1110px; height:570px;margin-top: 20px;">
						<div id="choice" v-for="(item, index) in choices" v-show="index + 1 == current">
							<h1>{{ index + 1 }}. {{ item.question }} [{{ item.score }}分]</h1>
							<el-radio-group v-model="answer[0][index]">
								<el-radio label="A" border>A. {{ item.answer_A }}</el-radio><br />
								<el-radio label="B" border>B. {{ item.answer_B }}</el-radio><br />
								<el-radio label="C" border>C. {{ item.answer_C }}</el-radio><br />
								<el-radio label="D" border>D. {{ item.answer_D }}</el-radio><br />
							</el-radio-group>
						</div>
						<div id="fill" v-for="(item, index) in fills" v-show="index + fillPos == current">
							<h1>{{ index + fillPos }}. {{ item.question }} [{{ item.score }}分]</h1>
							<el-input v-model="answer[1][index]"></el-input>
						</div>
						<div id="judge" v-for="(item, index) in judges" v-show="index + judgePos == current">
							<h1>{{ index + judgePos }}. {{ item.question }} [{{ item.socre }}分]</h1>
							<el-radio-group v-model="answer[2][index]">
								<el-radio label="T" border>正确</el-radio><br />
								<el-radio label="F" border>错误</el-radio><br />
							</el-radio-group>
						</div>
						<div id="program" v-for="(item, index) in programs" v-show="index + programPos == current">
							<h1>{{ index + programPos }}. {{ item.question }} [{{ item.score }}分]</h1>
							<el-row type="flex" justify="space-around">
								<el-checkbox-group v-model="answer_list[index].checkedValues">
									<el-checkbox v-for="option in answer_list[index].options" :key="option.label"
										:label="option.label">
										{{ option.label }}、{{ option.value }}
									</el-checkbox>
								</el-checkbox-group>
							</el-row>
						</div>
					</el-card>
				</el-main>
				<el-footer>
					<div class="box-div">
						<el-row type="flex" justify="start">
							<el-col :span="4" :offset="6">
								<el-button type="primary" @click="lastQuestion">上一题</el-button>
							</el-col>
							<el-col :span="4">
								<el-button type="primary" @click="nextQuestion">下一题</el-button>
							</el-col>
						</el-row>
					</div>
				</el-footer>
			</el-container>
		</el-container>
	</div>
</template>

<script>
import TimeCountDown from '@/components/TimeCountDown.vue'
import { ref } from 'vue'
import { MessageBox } from 'element-ui';
export default {
	data() {
		return {
			clazz: {},
			majorName: '',
			choices: [],
			fills: [],
			judges: [],
			programs: [],
			answer: [
				[],
				[],
				[],
				[]
			],
			numGroups:0,//复选题题目数
			answer_list:[],
			current: 1, //当前题号
			totalScore: 0, //总分
		}
	},
	created(){
		this.initGroups();
	},
	components: {
		TimeCountDown
	},
	computed: {
		//用计算属性显示结果
		fillPos() {
			return this.choices.length + 1;
		},
		judgePos() {
			return this.choices.length + this.fills.length + 1;
		},
		programPos() {
			return this.choices.length + this.fills.length + this.judges.length + 1;
		},
		totalNum() {
			return this.choices.length + this.fills.length + this.judges.length + this.programs.length;
		},
		getStudent() {
			return this.$store.state.student;
		},
		getUser() {
			return this.$store.state.user;
		},

		getPaper() {
			return localStorage.getItem('paper') ? JSON.parse(localStorage.getItem('paper')) : {};
		},
		getExam() {
			return localStorage.getItem('exam') ? JSON.parse(localStorage.getItem('exam')) : {};
		},
		getPractice() {
			return localStorage.getItem('practice') ? JSON.parse(localStorage.getItem('practice')) : {};
		},
		isPractice() {
			return this.$store.state.isPractice;
		}
	},
	methods: {
		isChecked(index) {
    // 假设answer_list[index]存在，并且checkedValues是一个数组
    const answerItem = this.answer_list[index];
    if (answerItem && Array.isArray(answerItem.checkedValues)) {
      // 检查checkedValues数组是否包含表示“选中”的值
      // 这里假设如果checkedValues数组非空，则表示该题目已被选中
      return answerItem.checkedValues.length > 0;
    }
    return false;
  },
		// 获取班级信息、专业的信息
		getClazzInfo() {
			this.$axios(`/api/clazzs/${this.getStudent.clazz}/?format=json`).then(res => {
				console.log(res.data);
				this.clazz = res.data;
				const majorId = res.data.major;
				// 使用majorId来获取专业名称
				this.$axios(`/api/majors/${majorId}/?format=json`).then(majorRes => {
					console.log(majorRes.data);
					// 假设 majorRes.data 包含专业名称的字段叫做 name
					this.majorName = majorRes.data.major_name;
				}).catch(majorError => {
					console.log(majorError);
				});
			}).catch(error => {
				console.log(error);
			})
		},
		getCourseId(){
			return localStorage.getItem('paper') ? JSON.parse(localStorage.getItem('paper')) : {};
		},
		// 获取选择题
		getChoiceInfo() {
			const courseId =this.getCourseId();
			this.$axios(`/api/choices/?format=json`, {
				params: {
					choice_number: this.getPaper.choice_number,
					level: this.getPaper.level,
					course_id : courseId.course
				}
			}).then(res => {
				this.choices = res.data
				console.log(this.choices)
			}).catch(error => {
				console.log(error)
			})
		},
		// 获取填空题
		getFillInfo() {
			const courseId =this.getCourseId();
			this.$axios('/api/fills/?format=json', {
				params: {
					fill_number: this.getPaper.fill_number,
					level: this.getPaper.level,
					course_id : courseId.course
				}
			}).then(res => {
				this.fills = res.data
				console.log(this.fills)
			}).catch(error => {
				console.log(error)
			})
		},
		// 获取判断题
		getJudgeInfo() {
			const courseId =this.getCourseId();
			this.$axios(`/api/judges/?format=json`, {
				params: {
					judge_number: this.getPaper.judge_number,
					level: this.getPaper.level,
					course_id : courseId.course
				}
			}).then(res => {
				this.judges = res.data
				console.log(this.judges)
			}).catch(error => {
				console.log(error)
			})
		},
		// 获取多选题
		getProgramInfo() {
			const courseId =this.getCourseId();
			this.answer_list = [];
			this.$axios(`/api/programs/?format=json`, {
				params: {
					program_number: this.getPaper.program_number,
					level: this.getPaper.level,
					course_id : courseId.course
				}
			}).then(res => {
				this.programs = res.data
				this.numGroups = res.data.length
				this.answer_list = this.programs.map(program => {
           		const options = program.answer_options.map((option, index) => {
                const label = String.fromCharCode(65 + index); // 生成 'A', 'B', 'C', ...
                return { label, value: option };
            });
            return {
                options,
                checkedValues: [] // 可以根据需要初始化checkedValues
            };
        });
				console.log(this.programs)
			}).catch(error => {
				console.log(error)
			})
		},
		// 下一题
		nextQuestion() {
			console.log(this.current, this.totalNum)
			if (this.current == this.totalNum) {
				this.$message("已经是最后一题了！")
			} else {
				this.current += 1
			}
		},
		// 上一题
		lastQuestion() {
			if (this.current == 1) {
				this.$message("已经是第一题了！")
			} else {
				this.current -= 1
			}
		},
		// 点击切换题目
		changeQuestion(number) {
			this.current = number
		},
		//阅卷
		goOverPaper() {
			this.totalScore = 0;
			console.log("1");
			this.goOverChoice();
			
			this.goOverFill();
			this.goOverJudge();
			
			this.goOverProgram();
		},
		//选择题阅卷
		goOverChoice() {
			for (let i = 0; i < this.choices.length; i++) {
				if (this.isPractice == true) {
					axios.post(`api/records/choices/`, {
						practice_id: this.getPractice.id,
						student_id: this.getStudent.id,
						choice_id: this.choices[i].id,
						your_answer: this.answer[0][i]
					}).then(res => {
						console.log(res); //处理成功的函数 相当于success

					}).catch(function (error) {
						console.log(error) //错误处理 相当于error
					});
				}
				//判断是否正确
				if (this.choices[i].right_answer == this.answer[0][i]) {
					this.totalScore += this.choices[i].score;
				}
			}
		},
		//填空题阅卷
		goOverFill() {
			for (let i = 0; i < this.fills.length; i++) {
				//如果是模拟练习，保存答题记录
				if (this.isPractice == true) {
					axios.post(`api/records/fills/`, {
						practice_id: this.getPractice.id,
						student_id: this.getStudent.id,
						fill_id: this.fills[i].id,
						your_answer: this.answer[1][i]
					}).then(res => {
						console.log(res); //处理成功的函数 相当于success

					}).catch(function (error) {
						console.log(error) //错误处理 相当于error
					});
				}
				//判断是否正确
				if (this.fills[i].right_answer == this.answer[1][i]) {
					this.totalScore += this.fills[i].score;
				}
			}
		},
		//判断题阅卷
		goOverJudge() {
			for (let i = 0; i < this.judges.length; i++) {
				//如果是模拟练习，保存答题记录
				if (this.isPractice == true) {
					axios.post(`api/records/judges/`, {
						practice_id: this.getPractice.id,
						student_id: this.getStudent.id,
						judge_id: this.judges[i].id,
						your_answer: this.answer[2][i]
					}).then(res => {
						console.log(res); //处理成功的函数 相当于success

					}).catch(function (error) {
						console.log(error) //错误处理 相当于error
					});
				}
				//判断是否正确
				if (this.judges[i].right_answer == this.answer[2][i]) {
					this.totalScore += this.judges[i].score;
				}
			}
		},
	
		goOverProgram() {
		for (let i = 0; i < this.programs.length; i++) {
			// 获取正确答案数组
			const correctAnswers = this.programs[i].right_answers.split(',').map(answer => answer.trim());
			// 假设 answer_list[i] 是一个对象，其中包含 checkedValues 属性，该属性是用户选择的答案数组
			const userAnswers = this.answer_list[i].checkedValues;
			
    		const isCorrect = correctAnswers.every(answer => userAnswers.includes(answer)) &&
                      userAnswers.length === correctAnswers.length;

			if (this.isPractice) {
			axios.post(`/api/records/programs/`, {
				practice_id: this.getPractice.id, // 确保属性名与数据中的一致
				student_id: this.getStudent.id, // 确保属性名与数据中的一致
				program_id: this.programs[i].id,
				your_answer: userAnswers.join(',') // 传递清理后的用户答案数组
			}).then(res => {
				console.log(res); // 处理请求成功的响应
			}).catch(error => {
				console.error(error); // 处理请求错误
			});
			}

			// 如果用户的答案是正确的，则加上该题的分数
			if (isCorrect) {
				this.totalScore += this.programs[i].score;
			}
			}
		},
		//交卷
		handIn() {
			this.goOverPaper();
			this.$axios(`/api/grade-records/?format=json`, {
				params: {
					exam_id: this.getExam.id,
					student_id: this.getStudent.id
				}
			}).then(res => {
				this.tempPagination = res.data;
				// 检查是否存在考试成绩，并且当前是考试模式
				if (this.tempPagination.results.length > 0 && !this.isPractice) {
					MessageBox.alert(`您已参加过考试，不能重复考试！您上次的考试成绩为: ${this.tempPagination.results[0].score} 分`, '提示', {
						confirmButtonText: '确定',
						callback: action => {
							this.$router.push('/');
						}
					});
				} else {
					// 如果是练习模式或者没有参加过考试，则正常处理
					clearTimeout(this.timer); // 清除延迟执行
					this.timer = setTimeout(() => { // 设置延迟执行
						if (this.isPractice) {
							// 如果是练习模式，则直接显示成绩并跳转到模拟练习
							this.$message({
								message: '你本次的测试成绩为：' + this.totalScore + '分',
								type: 'success'
							});
							this.$router.push('/practice');
						} else {
							// 如果是考试模式，则保存成绩并跳转到我的成绩页面
							this.$message({
								message: '你本次的考试成绩为：' + this.totalScore + '分',
								type: 'success'
							});
							axios.post(`api/grades/`, {
								"exam_id": this.getExam.id,
								"student_id": this.getStudent.id,
								"score": this.totalScore
							}).then(res => {
								this.$router.push('/grade');
							}).catch(error => {
								console.log(error);
							});
						}
					}, 1000);
				}
			}).catch(error => {
				console.log(error);
			})
		}
	},
	created() {
		this.getClazzInfo();
		this.getChoiceInfo();
		this.getFillInfo();
		this.getJudgeInfo();
		this.getProgramInfo();
	},
}
</script>

<style lang="scss" scoped>
.el-header {}

.el-aside {
	margin-left: 50px;
}

.el-main {}

.el-footer {}

.el-footer .el-row {
	margin-top: 10px;
}

#information {
	margin-top: 20px;
}

.box-div {
	border-radius: 2px;
}

.box-card {
	width: 250px;
	border-radius: 2px;
}

.question-tag {
	text-align: left;
}

.el-tag {
	margin-top: 5px;
	margin-left: 10px;
	width: 30px;
	height: 30px;
}

.bottom .el-tag {
	width: 20px;
	height: 20px;
}

#choice {
	float: left;
	width: auto;
	text-align: left;
}

#choice .el-radio-group {
	margin-left: 20px;
	text-align: left;
}

#choice .el-radio {
	margin-top: 40px;
}

#fill {
	float: left;
	width: auto;
	text-align: left;
}

#fill .el-input {
	margin-top: 20px;
	margin-left: 30px;
}

#judge {
	float: left;
	width: auto;
	text-align: left;
}

#judge .el-radio {
	margin-top: 40px;
}

#program {
	float: left;
	width: auto;
	text-align: left;
}
#program .el-checkbox-group {
	margin-left: 20px;
	text-align: left;
}

#program .el-checkbox {
	margin-top: 10px;
}

</style>
