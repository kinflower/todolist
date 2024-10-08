<template>
  <div class="container">
    <div class="title">
      <div>代办事项</div>
      <el-dropdown>
        <span class="el-dropdown-link" style="font-weight: bolder;cursor: pointer;color: #409EFF;">菜单</span>
        <template #dropdown>
          <el-dropdown-menu>
            <el-dropdown-item @click="todoCreate(todoForms)">新增</el-dropdown-item>
            <el-dropdown-item @click="setHide">{{ hide ? '显示' : '隐藏' }}已完成</el-dropdown-item>
          </el-dropdown-menu>
        </template>
      </el-dropdown>
    </div>

    <div style="display: flex;">
      <el-input v-model="todoKeyword" size="small" style="width: 100%;margin-right: 10px;" placeholder="输入事项名称"
        clearable @change="todoSearch"></el-input>
      <el-select v-model="todoKeyType" size="small" placeholder="选择待办类型" clearable @change="todoSearch">
        <el-option label="工作方面" value="工作方面"></el-option>
        <el-option label="学习方面" value="学习方面"></el-option>
        <el-option label="生活方面" value="生活方面"></el-option>
      </el-select>
    </div>
    <el-scrollbar :height="height + 'px'" style="margin-top: 10px; padding-right: 10px;">
      <div v-for="(item, index) in todoList" :key="index">
        <div class="todo_item">
          <el-checkbox v-model="item.state" @change="todoFinish(item)"><span class="grade">{{ item.grade
              }}</span><span :style="setTime(item) ? '' : 'color:red;'">{{ item.title
              }}</span></el-checkbox>
          <el-popover placement="left" trigger="hover" width="200px">
            <div>{{ item.detail ? item.detail : '未设置' }}</div>
            <template #reference>
              <el-icon>
                <MoreFilled class="el-icon-more" color="#8d8d8d" @click="handleEditTodo(item)" />
              </el-icon>
            </template>
          </el-popover>
        </div>
        <div class="line"></div>
      </div>
    </el-scrollbar>
    <el-drawer v-model="drawer" direction="ltr" size="100%" :title="edit ? '编辑' : '新增'">
      <el-form ref="todoForms" :model="todoForm" style="width: auto;margin: -20px 30px 0;">
        <el-form-item>
          <el-date-picker v-model="todoForm.start" type="datetime" clearable placeholder="选择开始时间" style="width: 100%;">
          </el-date-picker>
        </el-form-item>
        <el-form-item>
          <el-date-picker v-model="todoForm.end" type="datetime" clearable placeholder="选择结束时间" style="width: 100%;">
          </el-date-picker>
        </el-form-item>
        <el-form-item>
          <el-input v-model="todoForm.title" clearable placeholder="输入待办名称"></el-input>
        </el-form-item>
        <el-form-item>
          <div style="display: flex;width: 100%;">
            <el-select v-model="todoForm.type" clearable placeholder="选择分区类型">
              <el-option label="工作方面" value="工作方面"></el-option>
              <el-option label="学习方面" value="学习方面"></el-option>
              <el-option label="生活方面" value="生活方面"></el-option>
            </el-select>
            <el-select v-model="todoForm.grade" clearable placeholder="选择优先级">
              <el-option label="Ⅰ级" value="Ⅰ级"></el-option>
              <el-option label="Ⅱ级" value="Ⅱ级"></el-option>
              <el-option label="Ⅲ级" value="Ⅲ级"></el-option>
              <el-option label="Ⅳ级" value="Ⅳ级"></el-option>
            </el-select>
          </div>
        </el-form-item>
        <el-form-item>
          <el-input type="textarea" v-model="todoForm.detail" :autosize="{ minRows: 8, maxRows: 8 }"
            placeholder="输入待办详情"></el-input>
        </el-form-item>
        <el-form-item>
          <div style="text-align: right;width: 100%;">
            <el-button type="primary" size="small" @click="todoSave">保存</el-button>
            <el-button v-if="edit" type="danger" size="small" @click="todoRemove">删除</el-button>
          </div>
        </el-form-item>
      </el-form>
      <div class="finish" v-show="todoForm.finish">完成时间：{{ todoForm.finish }}</div>
    </el-drawer>
  </div>
</template>

<script setup>
import { MoreFilled } from "@element-plus/icons-vue";
import { ElMessage } from "element-plus";
import { computed, onBeforeMount, reactive, ref } from "vue";
import dayjs from "dayjs";
import request from "./api";
const drawer = ref(false)
const edit = ref(false)
const hide = ref(false)
const todoForm = reactive({
  id: '',
  title: '',
  state: false,
  detail: '',
  type: '',
  start: '',
  end: '',
  finish: '',
  grade: ''
})
const todoList = ref([])
const todoKeyword = ref("")
const todoKeyType = ref("")
const todoForms = ref(null)
const height = ref(0)

onBeforeMount(() => {
  getTodoList()
  height.value = window.innerHeight - 65
})

const setTime = computed(() => {
  return function (val) {
    if (!val.end) return true
    if (val.state) return true
    return parseInt(dayjs(val.end).format('DD')) >= parseInt(dayjs().format('DD'))
  }
})
function handleEditTodo(item) {
  todoForm.detail = item.detail
  todoForm.end = item.end
  todoForm.finish = item.finish
  todoForm.grade = item.grade
  todoForm.start = item.start
  todoForm.state = item.state
  todoForm.title = item.title
  todoForm.type = item.type
  todoForm.id = item.id
  edit.value = true
  drawer.value = true
}
function getTodoList() {
  request.get('/selectalltodo').then((res) => {
    let finList = []
    let unfinList = []
    res.msg.forEach(item => {
      item.state = item.state === 0 ? false : true
      if (item.state) {
        finList.push(item)
      } else {
        unfinList.push(item)
      }
    })
    if (hide.value) {
      todoList.value = unfinList
    } else {
      todoList.value = [...unfinList, ...finList]
    }
  }).catch((err) => ElMessage.error(JSON.stringify(err)))
}
function todoCreate() {
  todoForm.detail = ""
  todoForm.end = ""
  todoForm.finish = ""
  todoForm.grade = ""
  todoForm.start = ""
  todoForm.state = false
  todoForm.title = ""
  todoForm.type = ""
  todoForm.id = ""
  edit.value = false
  drawer.value = true
}
function todoSave() {
  if (!edit.value) {
    request.post('/inserttodo', JSON.stringify(todoForm)).then(() => {
      ElMessage({
        type: 'success',
        message: '保存成功!',
        showClose: true
      });
    }).catch(() => ElMessage.error('提交失败'))
  } else {
    request.post('/updatetodo', JSON.stringify(todoForm)).then(() => {
      ElMessage({
        type: 'success',
        message: '保存成功!',
        showClose: true
      });
    }).catch(() => ElMessage.error('提交失败'))
  }
  getTodoList()
  drawer.value = false
}
function todoRemove() {
  request.get(`/deletetodo?id=${todoForm.id}`).then(() => {
    ElMessage({
      type: 'success',
      message: '删除成功!',
      showClose: true
    });
    getTodoList()
    drawer.value = false
  }).catch(() => ElMessage.error('删除失败'))
}
function todoFinish(item) {
  item.finish = item.state ? dayjs().format('YYYY-MM-DD HH:mm:ss') : ''
  request.post('/updatetodo', JSON.stringify(item)).then(() => {
    ElMessage({
      type: 'success',
      message: item.state ? '事件已完成!' : '已取消',
      showClose: true
    });
    getTodoList()
  }).catch(() => ElMessage.error('提交失败'))
}
function todoSearch() {
  hide.value = false
  if (!todoKeyword.value && !todoKeyType.value) {
    getTodoList()
  } else {
    request.get(`/searchtodo?title=${todoKeyword.value}&type=${todoKeyType.value}`).then((res) => {
      let finList = []
      let unfinList = []
      res.msg.forEach(item => {
        item.state = item.state === 0 ? false : true
        if (item.state) {
          finList.push(item)
        } else {
          unfinList.push(item)
        }
      })
      todoList.value = [...unfinList, ...finList]
    }).catch((err) => ElMessage.error(JSON.stringify(err)))
  }
}
function setHide() {
  hide.value = !hide.value
  getTodoList()
}
</script>

<style scoped>
.container {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    padding: 0 10px 10px;
}

.title {
    display: flex;
    justify-content: space-between;
    margin: 5px;
    align-items: center;
}

.el_icon {
    padding: 4px 5px;
    color: #363a69;
    transition-duration: 500ms;
}

.el_icon:hover {
    cursor: pointer;
    background: #e6e6e6;
    transition-duration: 500ms;
}

.url_table {
    width: 100%;
}

.name {
    text-decoration: none;
    font-weight: bold;
    color: rgb(43, 128, 226);
}

.menu {
    position: fixed;
    top: 0;
    left: 0;
    bottom: 0;
    width: 180px;
}

.item {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin: 20px 0;
    width: 25%;
    cursor: pointer;
}

.title {
    color: #666666;
    font-weight: bold;
    font-size: 16px;
    margin-top: 5px;
}

.lnk {
    display: flex;
    flex-wrap: wrap;
}

.icon {
    width: 80px;
    height: 80px;
    color: #ffffff;
    text-shadow: 2px 1px 2px rgb(138, 120, 120);
    border-radius: 10px;
    display: flex;
    justify-content: center;
    align-items: center;
    overflow: hidden;
    white-space: nowrap;
}

.key_search {
    display: block;
    width: 250px;
    margin: 20px auto 100px;
}

.header {
    position: absolute;
    top: 15px;
    right: 15px;
    text-align: right;
    z-index: 1;
}

.study {
    text-decoration: none;
    text-align: right;
    color: #8d8d8d;
}

.todo_item {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.grade {
    font-size: 12px;
    color: #8d8d8d;
    margin: 0 5px 0 -5px;
}

.el-icon-more,
.el-icon-edit {
    cursor: pointer;
}

.line {
    border-bottom: 1px solid #d8d8d8;
    margin: 5px 0;
}

.finish {
    position: absolute;
    right: 10px;
    bottom: 5px;
    font-size: 14px;
    color: #8d8d8d;
}
</style>