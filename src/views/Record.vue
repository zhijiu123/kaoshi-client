<template>
	<div id="record">
		<div id="choice-record" class="type">
			<h2>选择题</h2>
			<div class="question" v-for="(item,index) in choiceRecord">
				<h4>{{index + 1}}. {{item.choice.question}}</h4>
				<el-radio-group v-model="item.your_answer" disabled>
					<el-radio label="A">A. {{item.choice.answer_A}}</el-radio><br />
					<el-radio label="B">B. {{item.choice.answer_B}}</el-radio><br />
					<el-radio label="C">C. {{item.choice.answer_C}}</el-radio><br />
					<el-radio label="D">D. {{item.choice.answer_D}}</el-radio><br />
				</el-radio-group>
				<h4>结果:
					<el-tag :type="item.choice.right_answer === item.your_answer ? 'success' : 'danger'">
						{{item.choice.right_answer === item.your_answer ? "正确" : "错误"}}
					</el-tag>
				</h4>
				<h4>正确答案:{{item.choice.right_answer}}</h4>
				<h4>分数:{{item.choice.score}}</h4>
				<h4>难度:<el-rate v-model="item.choice.level" disabled="" style="margin-left: 10px;"></el-rate>
				</h4>
				<h4>解析:{{item.choice.analysis}}</h4>
			</div>
		</div>
		<div id="fill-record" class="type">
			<h2>填空题</h2>
			<div class="question" v-for="(item,index) in fillRecord">
				<h4>{{index + 1}}. {{item.fill.question}}</h4>
				<el-input v-model="item.your_answer" disabled></el-input>
				<h4>结果:
					<el-tag :type="item.fill.right_answer === item.your_answer ? 'success' : 'danger'">
						{{item.fill.right_answer === item.your_answer ? "正确" : "错误"}}
					</el-tag>
				</h4>
				<h4>正确答案:{{item.fill.right_answer}}</h4>
				<h4>分数:{{item.fill.score}}</h4>
				<h4>难度:<el-rate v-model="item.fill.level" disabled="" style="margin-left: 10px;"></el-rate>
				</h4>
				<h4>解析:{{item.fill.analysis}}</h4>
			</div>
		</div>
		<div id="judge-record" class="type">
			<h2>判断题</h2>
			<div class="question" v-for="(item,index) in judgeRecord">
				<h4>{{index + 1}}. {{item.judge.question}}</h4>
				<el-radio-group v-model="item.your_answer" disabled>
					<el-radio label="T" border>正确</el-radio><br />
					<el-radio label="F" border>错误</el-radio><br />
				</el-radio-group>
				<h4>结果:
					<el-tag :type="item.judge.right_answer === item.your_answer ? 'success' : 'danger'">
						{{item.judge.right_answer === item.your_answer ? "正确" : "错误"}}
					</el-tag>
				</h4>
				<h4>正确答案:{{item.judge.right_answer}}</h4>
				<h4>分数:{{item.judge.score}}</h4>
				<h4>难度:<el-rate v-model="item.judge.level" disabled="" style="margin-left: 10px;"></el-rate>
				</h4>
				<h4>解析:{{item.judge.analysis}}</h4>
			</div>
		</div>
		<div id="program-record" class="type">
			<h2>多选题</h2>
			<div class="question" v-for="(item, index) in programRecord">
				<h4>{{ index + 1 }}. {{ item.program.question }}</h4>

				<el-checkbox-group v-model="item.your_answer_list" disabled>
					<el-checkbox v-for="option in answer_list[index].options" :key="option.label" :label="option.label">
						{{ option.label }}、{{ option.option }}
					</el-checkbox>
				</el-checkbox-group>
				<h4>结果:
					<el-tag :type="checkAnswers(item.program.right_answers, item.your_answer) ? 'success' : 'danger'">
						{{ checkAnswers(item.program.right_answers, item.your_answer) ? "正确" : "错误" }}
					</el-tag>
				</h4>
				<h4>你的答案：{{ item.your_answer }}</h4>
				<h4>正确答案: {{ item.program.right_answers }}</h4>
				<h4>分数: {{ item.program.score }}</h4>
				<h4>难度: <el-rate v-model="item.program.level" disabled="" style="margin-left: 10px;"></el-rate></h4>
				<h4>解析: {{ item.program.analysis }}</h4>
			</div>
		</div>
	</div>
</template>

<script>
	export default {
		data() {
			return {
				choiceRecord: [], //选择题练习记录
				fillRecord: [], //填空题练习记录
				judgeRecord: [], //判断题练习记录
				programRecord: [], //多选题练习记录
				answer_list:[]
			}
		},
		created() {
			this.getChoiceRecordInfo();
			this.getFillRecordInfo();
			this.getJudgeRecordInfo();
			this.getProgramRecordInfo();
		},
		methods: {
			trimYourAnswer(yourAnswer){
				return   yourAnswer.split(',').map(answer => answer.trim());
			},
			checkAnswers(rightAnswers, yourAnswer) {
			// 将字符串转换为数组，并去除每个元素的空格
			const rightAnswerArray = rightAnswers.split(',').map(answer => answer.trim());
			const yourAnswerArray = yourAnswer.split(',').map(answer => answer.trim());

			// 检查每个元素是否都在对方的数组中
			return rightAnswerArray.every(answer => yourAnswerArray.includes(answer)) &&
					yourAnswerArray.every(answer => rightAnswerArray.includes(answer));
			},
			//获取选择题练习记录
			getChoiceRecordInfo() {
				this.$axios(`api/records/choices/?format=json`, {
					params: {
						page: this.page,
						page_size: this.page_size,
						practice_id: this.$route.query.practice_id,
					}
				}).then(res => {
					this.loading = false
					this.choiceRecord = res.data
				}).catch(error => {
					console.log(error)
				})
			},
			//获取选择题练习记录
			getFillRecordInfo() {
				this.$axios(`api/records/fills/?format=json`, {
					params: {
						page: this.page,
						page_size: this.page_size,
						practice_id: this.$route.query.practice_id,
					}
				}).then(res => {
					this.loading = false
					this.fillRecord = res.data
				}).catch(error => {
					console.log(error)
				})
			},
			//获取判断题练习记录
			getJudgeRecordInfo() {
				this.$axios(`api/records/judges/?format=json`, {
					params: {
						page: this.page,
						page_size: this.page_size,
						practice_id: this.$route.query.practice_id,
					}
				}).then(res => {
					this.loading = false
					this.judgeRecord = res.data
				}).catch(error => {
					console.log(error)
				})
			},
			//获取多选题练习记录
			getProgramRecordInfo() {
				this.$axios(`api/records/programs/?format=json`, {
					params: {
						page: this.page,
						page_size: this.page_size,
						practice_id: this.$route.query.practice_id,
					}
				}).then(res => {
					this.loading = false
					this.programRecord = res.data;
					this.answer_list = this.programRecord.map(record => {
						const options = record.program.answer_options.map((option, index) => {
							const label = String.fromCharCode(65 + index); // 将索引转换为大写字母标签
							return { label, option };
						});
						const valuesChecked = record.your_answer_list;
						return { options, valuesChecked };
					});
				}).catch(error => {
					console.log(error)
				})
			},
		}
	}
</script>

<style lang="scss" scoped>
#record {
//   width: 80%;
//   margin: 0 auto;
  background-color: #f4f4f4; // 轻柔的背景颜色
  padding: 20px;
  font-family: 'Arial', sans-serif; // 更易读的字体
}

.type {
  background-color: #fff; // 明亮的背景颜色
  border-radius: 10px; // 圆角边框
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); // 轻微的阴影
  margin-bottom: 20px;
  padding: 15px;
}

h2 {
  color: #333; // 深色标题
  font-weight: bold;
  margin-bottom: 10px;
}

.question {
  text-align: left;
  margin-left: 0; // 移除不必要的左边距
  border-bottom: 1px solid #eee; // 添加分隔线
  padding: 10px 0;
  
  &:last-child {
    border-bottom: none; // 移除最后一个问题的分隔线
  }
}

h4 {
  color: #666; // 柔和的文字颜色
  margin: 10px 0;
}

.el-radio-group, .el-checkbox-group {
  margin: 10px 0;
}

.el-radio, .el-checkbox {
  display: block; // 使单选框和复选框各自占一行
  margin: 5px 0;
}

.el-tag {
  margin: 5px;
}

.el-input {
  max-width: 800px;
  margin: 10px 0;
}

.el-rate {
  margin-top: 10px;
  display: inline-block; // 修复评分组件的布局
}

// 响应式设计
@media (max-width: 768px) {
  .question {
    margin-left: 0;
  }
  
  .el-rate {
    margin-right: 0;
  }
}
</style>
