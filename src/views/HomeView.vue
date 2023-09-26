<template>
	<div style="padding: 10px;">
		<a-button type="primary" @click="handleGenerator">生成</a-button>
		<a-button style="margin: 0 10px;" @click="handleDownload">下载</a-button>
	
		<p style="margin-top: 20px;"></p>
	
		<a-table
			class="match_table"
			:dataSource="dataSource" 
			:columns="columns" 
			:pagination="false"
			@resizeColumn="handleResizeColumn"
		>
			<template #bodyCell="{ column, record, index  }">
				<template v-if="column.key === 'matchNumStr'">
					<p @dblclick="dataSource.splice(index, 1)" @touchstart="dataSource.splice(index, 1)">{{ record.businessDate + ' / ' + record.matchNumStr }}</p>
				</template>
	
				<template v-if="column.key === 'homeTeamAllName'">
					<span style="margin-right: 5px;color: red;">{{ record.hhad.goalLine }}</span>
					<span @dblclick="record.edit=true;" @touchstart="record.edit=true;">{{ record.homeTeamAllName }} VS {{ record.awayTeamAllName }}</span>
				</template>
	
				<template v-if="column.key === 'rang'">
					<span>{{ record.homeTeamAllName }} VS {{ record.awayTeamAllName }}</span>
				</template>
	
				<template v-if="column.key === 'hhad'">
					<div class="multiple_wrapper">
						<span class="had" @click="handleSpan">{{ record.had.h || $nullText }}</span>
						<span class="had" @click="handleSpan">{{ record.had.d || $nullText }}</span>
						<span class="had" @click="handleSpan">{{ record.had.a || $nullText }}</span>
					</div>
					<div class="multiple_wrapper">
						<em class="hhad_tip">让</em>
						<span class="hhad" @click="handleSpan">{{ record.hhad.h || $nullText }}</span>
						<span class="hhad" @click="handleSpan">{{ record.hhad.d || $nullText }}</span>
						<span class="hhad" @click="handleSpan">{{ record.hhad.a || $nullText }}</span>
					</div>
				</template>
	
				<template v-if="column.key === 'prediction'">
					<div v-if="record.edit" class="text_wrapper">
						<a-input v-model:value="record.prediction" @blur="record.edit=false;" />
					</div>
	
					<div v-else class="text_wrapper">
						{{ record.prediction || $nullText }}
					</div>
				</template>
			</template>
		</a-table>
	</div>
</template>

<script setup>
import { ref } from 'vue';
import { Button as AButton, Table as ATable, Input as AInput } from 'ant-design-vue';
import html2canvas from 'html2canvas';

const dataSource = ref([]);
const columns = ref([
	{
		title: '编号',
		dataIndex: 'matchNumStr',
		key: 'matchNumStr',
		width: 200,
		align: 'center',
		resizable: true,
	},
	{
		title: '赛事',
		dataIndex: 'homeTeamAllName',
		key: 'homeTeamAllName',
		resizable: true,
		align: 'center',
	},
	{
		title: '预测胜负平',
		dataIndex: 'hhad',
		key: 'hhad',
		resizable: true,
		align: 'center',
	},
	{
		title: '其他预测',
		dataIndex: 'prediction',
		key: 'prediction',
		align: 'center',
	},
]);
const today = new Date().toLocaleDateString().split('/').map(item => item >= 10 ? item : '0' + item).join('-');
const $nullText = '--';

function handleGenerator() {
	fetch('https://webapi.sporttery.cn/gateway/jc/football/getMatchCalculatorV1.qry?poolCode=hhad,had&channel=c', {
	}).then(res => {
		res.json().then(res2 => {
			if (res2?.value?.matchDateList?.length) {
				const data = res2.value;
				const todayMatch = data.matchDateList.find(item => item.businessDate === today);
				if (todayMatch) {
					const matchObj = data.matchInfoList.find(item => item.weekday === todayMatch.businessDateCn);
					dataSource.value = matchObj.subMatchList.map((item, index) => {
						return {
							...item,
							key: item.businessDate + index,
						}
					});
				}
			}
		})
	})
}

function handleDownload () {
	html2canvas(document.querySelector(".match_table"), { dpi: 300, scale: 2 }).then(canvas => {
		const img = new Image();
		img.src = canvas.toDataURL('image/jpeg', 1.0);
		img.onload = function() {
			const a = document.createElement('a');
			a.download = today + '赛事.jpg';
			a.href = img.src;
			a.click();
			a.remove();
		}
	});
}

function handleResizeColumn(w, col) {
  col.width = w;
}

function handleSpan({ target }) {
	if (target.innerText === $nullText) {
		return;
	}
	if (!target.style.background) {
		target.style.background = 'red';
		target.style.color = '#fff';
	} else {
		target.style.background = '';
		target.style.color = '';
	}
}
</script>

<style lang="scss" scoped>
	.text_wrapper {
		width: 100%;
		height: 30px;
	}

	.multiple_wrapper {
		position: relative;
		display: flex;
		height: 100%;
		span {
			display: inline-block;
			width: 33%;
			min-height: 30px;
			text-align: center;
			line-height: 30px;
			border: 1px solid #F0F0F0;
			box-sizing: border-box;
			cursor: pointer;
		}
		.had {
			border-bottom: none;
			border-top: none;
		}
		.hhad {
			border-bottom: none;
			background-color: #f8f8f8;
		}
		.hhad_tip {
			position: absolute;
			left: -1.5em;
			font-size: 12px;
			line-height: 30px;
			color: red;
		}
		span:nth-child(1), span:nth-child(2) {
			border-right: none;
		}
	}

	:deep(td.ant-table-cell:nth-child(3)) {
		padding: 0;
	}
</style>
