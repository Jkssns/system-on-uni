<template>
	<a-button type="primary" @click="handleGenerator">生成</a-button>
	<a-button style="margin: 0 10px;" @click="handleDownload">下载</a-button>

	<p style="margin-top: 20px;"></p>

	<a-table
		class="math_table" 
		:dataSource="dataSource" 
		:columns="columns" 
		:pagination="false" 
		bordered
	>
		<template #bodyCell="{ column, record, index  }">
			<template v-if="column.key === 'matchNumStr'">
				<span @dblclick="dataSource.splice(index, 1)" @touchstart="dataSource.splice(index, 1)">{{ record.matchNumStr }}</span>
			</template>

			<template v-if="column.key === 'homeTeamAllName'">
				<span @dblclick="record.edit=true;" @touchstart="record.edit=true;">{{ record.homeTeamAllName }} VS {{ record.awayTeamAllName }}</span>
			</template>

			<template v-if="column.key === 'prediction'">
				<div v-if="record.edit" class="text_wrapper">
					<a-input v-model:value="record.prediction" @blur="record.edit=false;" />
				</div>

				<div v-else class="text_wrapper">
					{{ record.prediction || '' }}
				</div>
			</template>
		</template>
	</a-table>
</template>

<script setup>
import { ref } from 'vue';
import { Button as AButton, Table as ATable, Input as AInput } from 'ant-design-vue';
import html2canvas from 'html2canvas';

const dataSource = ref([]);
const columns = [
	{
		title: '编号',
		dataIndex: 'matchNumStr',
		key: 'matchNumStr',
		width: '100px',
	},
	{
		title: '赛事',
		dataIndex: 'homeTeamAllName',
		key: 'homeTeamAllName',
	},
	{
		title: '预测',
		dataIndex: 'prediction',
		key: 'prediction',
	},
];
const today = new Date().toLocaleDateString().split('/').map(item => item >= 10 ? item : '0' + item).join('-');

function handleGenerator() {
	fetch('https://webapi.sporttery.cn/gateway/jc/football/getMatchListV1.qry?clientCode=3001', {
	}).then(res => {
		res.json().then(res2 => {
			if (res2?.value?.matchDateList?.length) {
				const data = res2.value;
				const todayMatch = data.matchDateList.find(item => item.businessDate === today);
				if (todayMatch) {
					const matchObj = data.matchInfoList.find(item => item.weekday === todayMatch.businessDateCn);
					dataSource.value = matchObj.subMatchList;
				}
			}
		})
	})
}

function handleDownload () {
	html2canvas(document.querySelector(".math_table"), { dpi: 300, scale: 2 }).then(canvas => {
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
</script>

<style lang="scss" scoped>
	.text_wrapper {
		width: 100%;
		height: 30px;
	}
</style>
