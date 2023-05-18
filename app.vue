<template>
  <!-- Choose company dropdown -->
  <el-dropdown size="large" split-button type="primary">
    {{ companyDropDownText }}
    <template #dropdown>
      <el-dropdown-menu>
        <el-dropdown-item v-for="com in fetchedCompanies" :key="com" @click="handleCompanyChange(com)"> {{ com }} </el-dropdown-item>
      </el-dropdown-menu>
    </template>
  </el-dropdown>

  <br />
  <br />

  <!-- Choose timeline dropdown -->
  <el-dropdown size="large" split-button type="primary">
    {{ timelineDropDownText }}
    <template #dropdown>
      <el-dropdown-menu>
        <el-dropdown-item v-for="t in fetchedTimelines" :key="t" @click="handleTimelineChange(t)"> {{ t }} </el-dropdown-item>
      </el-dropdown-menu>
    </template>
  </el-dropdown>

  <br />
  <br />

  <!-- Show the table -->
  <el-table
    :data="fetchedQuestions[0]"
    style="width: 100%"
    :row-class-name="getSolvedStatus"
  > 
    <el-table-column label="Status" width="120" >
      <template #default="scope">
        <el-button @click="changeQuestionStatus(scope.row.leetcode_id)">Toggle</el-button>
      </template>
    </el-table-column>
    <el-table-column prop="leetcode_id" label="ID" width="60" />
    <el-table-column label="Name" width="600" >
      <template #default="scope">
        <div style="display: flex; align-items: center">
          <a :href="scope.row.link" target="__blank" style="margin-left: 10px">{{ scope.row.title }}</a>
        </div>
      </template>
    </el-table-column>
    <el-table-column prop="acceptance" label="Acceptance" />
    <el-table-column prop="difficulty" label="Rating" />
    <el-table-column prop="frequency" label="Frequency" />

  </el-table>
</template>

<script setup>
import nuxtStorage from 'nuxt-storage';

useHead({
  title: 'Leetcode by Companies',
})
    
const runtimeConfig = useRuntimeConfig();

const companyDropDownText = ref("Choose a company!")
const timelineDropDownText = ref("Choose a timeline!")
const company = ref("");
const timeline = ref("");

// Load the solved questions on intial load from the local storage
const loadData = nuxtStorage.localStorage.getData('solvedQuestions');
if (loadData == null) {
  let m = {};
  for (let i = 1; i <= 2600; i++) m[i] = false;
  nuxtStorage.localStorage.setData('solvedQuestions', m, 365, "d");
};
const solvedData = nuxtStorage.localStorage.getData('solvedQuestions');
const solvedQuestionsLocalStorage = ref(solvedData);

const { data: fetchedCompanies } = await useLazyFetch('/company', {baseURL: runtimeConfig.public.API_BASE_URL});
const { data: fetchedTimelines } = await useLazyFetch('/time', {
    method: "POST",
    body: {
      company: company
    },
    baseURL: runtimeConfig.public.API_BASE_URL
})
const { data: fetchedQuestions } = await useLazyFetch('/questions', {
    method: "POST",
    body: {
      company: company,
      timeline: timeline
    },
    baseURL: runtimeConfig.public.API_BASE_URL
})

const handleCompanyChange = (com) => {
  company.value = com;
  companyDropDownText.value = com;
  timeline.value = "";
  timelineDropDownText.value = "Choose a timeline!";
}

const handleTimelineChange = (t) => {
  timeline.value = t;
  timelineDropDownText.value = t;
}

const getSolvedStatus = ({
  row
}) => {
  if (solvedQuestionsLocalStorage.value[row.leetcode_id])
    return 'success-row'
  return ''
}

const changeQuestionStatus = (id) => {
  solvedQuestionsLocalStorage.value[id] = !solvedQuestionsLocalStorage.value[id];
  nuxtStorage.localStorage.setData('solvedQuestions', solvedQuestionsLocalStorage.value, 365, "d");
}

</script>

<style>
.el-table .warning-row {
  --el-table-tr-bg-color: var(--el-color-warning-light-9);
}
.el-table .success-row {
  --el-table-tr-bg-color: var(--el-color-success-light-9);
}
</style>