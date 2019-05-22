<template>
  <div id="app">
    <nav class="todo-nav">
      <div class="bg-primary">
        <div class="container">
          <ul class="list-unstyled mb-0 d-flex justify-content-between nav-menu"
          @click="gettodos"> <!-- 冒泡觸發-->
            <li class="py-4 px-3" :class="{active : visibility === 'all'}"
            @click="visibility = 'all' ">My Tasks</li>
            <li class="py-4 px-3" :class="{active : visibility === 'undone'}"
            @click="visibility = 'undone' ">In Progress</li>
            <li class="py-4 px-3" :class="{active : visibility === 'done' }"
            @click="visibility = 'done' ">Completed</li>
          </ul>
        </div>
      </div>
    </nav>

    <section class="todo-content">
      <div class="container py-4">
        <h2 class="h4 mt-3 mb-5 text-center text-danger"
        v-if="visibility === 'done' && done === 0">
          沒有完成任何一件
        </h2>
        <h2 class="h4 mt-3 mb-5 text-center text-info"
        v-if="visibility === 'undone' && total - done === 0">
          已完成全部事項
        </h2>
        <h2 class="h4 mt-3 mb-5 text-center text-dark"
        v-if="visibility === 'all' && todos.length === 0">
          開始紀錄今日事項吧
        </h2>
        <div v-if="visibility === 'all'">
          <input type="text" class="form-control addTask mb-4" placeholder="+ Add New"
        v-if="!isNew" @click="addtodo">
       </div>
        <div class="todo-edit mb-3 bg-light" v-if="isNew">
         <div class="edit-title p-4 d-flex align-items-center justify-content-between"
         :class="{'bg-warnlight': temtodos.star} ">
           <input type="text" class="form-control" placeholder="請輸入內容"
           v-model="temtodos.thing" @blur="leavetitle" v-if="!leaveinput">
            <div class="d-flex align-items-center" v-if="leaveinput" >
              <input type="checkbox" id="thecheckbox"
              @click="updatetemtodo" v-if="isedit"  v-model="temtodos.status">
              <h3 class="title mb-0 ml-2">{{temtodos.thing}}</h3>
            </div>
            <div class="title-icon" v-if="leaveinput"  @click="updatetemtodo"> <!-- 利用冒泡觸發 -->
              <font-awesome-icon :icon="['far','star']"  class="mr-3 star" v-if="!temtodos.star"
              @click.prevent="temtodos.star = !temtodos.star"/>
              <font-awesome-icon icon="star"  class="mr-3 text-warning fullstar" v-else
              @click.prevent="temtodos.star = !temtodos.star"/>
              <a href="#" @click="leaveinput = false">
                <font-awesome-icon icon="pencil-alt" />
              </a>
            </div>
          </div>
          <div class="edit-body px-7 py-4">
            <div class="mb-3">
              <h4>
              <font-awesome-icon :icon="['far','calendar-alt']" class="mr-1" />
                Deadline
              </h4>
              <input type="date" class="form-control" v-model="temtodos.deadline">
            </div>
            <div class="mb-3">
              <h4>
                <font-awesome-icon :icon="['far','file']" class="mr-1" />
                File
              </h4>
              <label for="files">
                <img src="../src/assets/＋.png" width="24px">
              </label>
              <input type="file" id="files" class="files">
            </div>
            <div>
              <h4>
                <font-awesome-icon :icon="['far','comment-dots']" class="mr-1" />
                Comment
              </h4>
              <textarea class="form-control" id="exampleFormControlTextarea1" rows="3"
              v-model="temtodos.comment"></textarea>
            </div>
          </div>
          <div class="edit-footer bg-white">
            <div class="row no-gutters">
              <div class="col-6">
                <button class="btn btn-block rounded-0 text-danger py-2 cancel"
                @click="cancelsave">
                  X
                  Cancel
                </button>
              </div>
              <div class="col-6">
                <button class="btn btn-block btn-primary rounded-0 py-2" @click="savetodo">
                  +
                  Save
                </button>
              </div>
            </div>
          </div>
        </div>

        <draggable v-model="todos" @end="updatesort">
          <div v-for="item in filteredTodos" :key="item.id">
            <div class="todo-item bg-light p-4 mb-2"
            :class="{'bg-warnlight': item.star} " v-if="item.id !== temtodos.id">
              <div class="d-flex align-items-center mb-1 justify-content-between">
                <div class="d-flex align-items-center">
                  <input type="checkbox" id="thecheckbox" v-model="item.status"
                  @click="updatetemtodo">
                  <h3 class="title mb-0 ml-2" :class="{active : item.status}">{{item.thing}}</h3>
                </div>
                <div class="title-icon" @click="updatetemtodo"> <!-- 利用冒泡觸發 -->
                  <font-awesome-icon :icon="['far','star']"  class="mr-3 star" v-if="!item.star"
                  @click.prevent="item.star = !item.star"/>
                  <font-awesome-icon icon="star"  class="mr-3 text-warning fullstar" v-else
                  @click.prevent="item.star = !item.star"/>
                  <a href="#" class="mr-3 pencil" @click.prevent="edittodo(item)">
                    <font-awesome-icon icon="pencil-alt" />
                  </a>
                  <a href="#" class="text-secondary"
                  @click.prevent="removetemtodo(item)"><font-awesome-icon icon="trash-alt" /></a>
                </div>
              </div>
              <div class="small-icon d-flex">
                <div class="mr-3 d-flex align-items-center" v-if="item.deadline">
                  <font-awesome-icon :icon="['far','calendar-alt']" class="mr-1" />
                  <span class="deadline">{{item.deadline}}</span>
                </div>
                <div class="mr-3 d-flex align-items-center" v-if="item.files">
                  <font-awesome-icon :icon="['far','file']" class="mr-1" />
                </div>
                <div class="d-flex align-items-center" v-if="item.comment">
                  <font-awesome-icon :icon="['far','comment-dots']" class="mr-1" />
                  <span>{{item.comment}}</span>
                </div>
              </div>
            </div>
          </div>
        </draggable>

        <div class="d-flex justify-content-around statistics">
          <span>Total {{total}}</span>
          <span>Done {{done}}</span>
          <span>Undone {{total - done}}</span>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import draggable from 'vuedraggable';

export default {
  name: 'app',
  components: {
    draggable,
  },
  data() {
    return {
      todos: [],
      isNew: false, // 切換編輯跟新增框
      isedit: false, // 是否編輯中
      leaveinput: false, // 是否離開input
      temtodos: {
        thing: '請輸入事項',
        deadline: '',
        comment: '',
        id: '',
        status: false,
        star: false,
      },
      visibility: 'all', // 當前區域
      total: 0,
      done: 0,
      drag: false,
    };
  },
  methods: {
    gettodos() {
      const array = JSON.parse(localStorage.getItem('vuetodos'));
      const sorttodos = [];
      if (array.length !== 0) {
        this.drag = true;
      }
      if (!this.drag || !array.every(item => item.order)) {
        array.sort((a, b) => (a.id > b.id ? 1 : -1));
      }
      if (array.some(item => item.order)) {
        array.sort((a, b) => (a.order > b.order ? 1 : -1));
      }

      const stararray = array.filter(item => item.star === true);
      const nostararray = array.filter(item => item.star === false);
      sorttodos.push(...stararray);
      sorttodos.push(...nostararray);

      this.total = sorttodos.length;
      this.done = sorttodos.filter(item => item.status === true).length;

      if (Object.keys(sorttodos).length !== 0) {
        this.todos = [...sorttodos];
      }
    },
    addtodo() {
      this.cleartemtodo();
      this.isedit = false;
      this.isNew = true;
    },
    updatetemtodo() {
      setTimeout(() => {
        localStorage.setItem('vuetodos', JSON.stringify(this.todos));
        this.gettodos();
      }, 100);
    },
    edittodo(item) {
      this.temtodos = item;
      this.isNew = true;
      this.isedit = true;
    },
    savetodo() {
      const vm = this;
      const times = Date.now();
      if (!vm.isedit) {
        vm.temtodos.id = times;
        if (vm.temtodos.thing.trim() && vm.temtodos.thing !== '請輸入事項') {
          vm.todos.push(vm.temtodos);
          vm.isNew = false;
          vm.leaveinput = false;
        }
      } else {
        vm.isNew = false;
        vm.leaveinput = false;
        vm.isedit = false;
        vm.todos.forEach((item, i) => {
          if (item.id === vm.temtodos.id) {
            vm.todos[i] = vm.temtodos;
          }
        });
      }
      localStorage.setItem('vuetodos', JSON.stringify(vm.todos));
      vm.cleartemtodo();
      vm.gettodos();
    },
    cancelsave() {
      if (this.isedit) {
        this.isedit = false;
        this.isNew = false;
        this.leaveinput = false;
        this.cleartemtodo();
        this.gettodos();
      } else {
        this.isNew = false;
      }
    },
    cleartemtodo() {
      this.temtodos.thing = '請輸入事項';
      this.temtodos.deadline = '';
      this.temtodos.comment = '';
      this.temtodos.id = '';
      this.temtodos.status = false;
      this.temtodos.star = false;
    },
    removetemtodo(item) {
      this.todos.splice(this.todos.indexOf(item), 1);
      localStorage.setItem('vuetodos', JSON.stringify(this.todos));
    },
    leavetitle() {
      if (this.temtodos.thing.trim() && this.temtodos.thing !== '請輸入事項') {
        this.leaveinput = true;
      } else {
        this.temtodos.thing = '請輸入事項';
      }
    },
    updatesort() {
      this.drag = true;
      this.todos.forEach((item, i) => {
        // eslint-disable-next-line no-param-reassign
        item.order = i;
      });
      this.updatetemtodo();
    },
  },
  computed: {
    filteredTodos() {
      const vm = this;
      let array = [];
      if (vm.visibility === 'done') {
        array = vm.todos.filter(item => item.status === true);
      }
      if (vm.visibility === 'undone') {
        array = vm.todos.filter(item => item.status === false);
      }
      if (vm.visibility === 'all') {
        array = [...this.todos];
      }
      return array;
    },
  },
  created() {
    this.gettodos();
  },
};
</script>

<style lang="scss">
  @import './assets/all';
</style>
