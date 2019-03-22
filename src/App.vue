<template>
  <div id="app">
    <div class="container">
      <input type="text" v-model="inputTask" @keyup.enter="addTask" />
      <input id="toggle-all" class="toggle-all" type="checkbox" v-model="allDone">
    </div>    
    <div v-for="(todo,index) in todolist" :key="todo.id" class="taskList" :class="{check: todo.checked}" v-show="displayByState(todo)"> 
         <div class="view" v-show="!todo.isEditing">
            <input type="checkbox" v-model="todo.checked" @change="checkedOne(index)">
            <p class="text" @dblclick="toEdit(todo)">{{todo.text}}</p>
            <span @click="deleOne(index)">x</span> 
         </div>
         <input class="edit" type="text" v-model="todo.text" v-show="todo.isEditing" v-todo-focus="todo.text" @blur="unEdit(todo,index)" @keyup.enter="unEdit(todo,index)" @keyup.esc="cancelEdit(todo)">
    </div>
    <div class="flex">
        <div><strong>{{ remainingCount }}</strong> 个未完成</div>
        <div class="filters">
          <a @click="changeState(0)" >全部</a>
          <a @click="changeState(1)" >激活</a>
          <a @click="changeState(2)"  >完成</a>
        </div>
        <div v-show="todolist.some( todo => todo.checked)" @click="clearDone()">清除已完成</div>
    </div>
  </div>
</template>

<script>
  
  
export default {
  name: 'app',
  data () {
    return {
      inputTask: '',
      todolist:JSON.parse(window.localStorage.getItem('todolist') || '[]'),
      filterText: 'all',
      state:0,
      // editedTodo:null
     
    }
  },
  mounted() {
    // 路由状态切换
    // window.onhashchange= () => {
    //   console.log(window.location.hash)
    //     var hash=window.location.hash.substring(1) || 'all';
    //     console.log(hash)
    //     this.filterText=hash;
    // };
    // 页面第一次进来,保持状态
    // window.onhashchange();
  },
  filters : {
		all: function (todolist) {
			return todolist;
		},
		active: function (todolist) {
			return todolist.filter(function (todo) {
				return !todo.checked;
			});
		},
		completed: function (todolist) {
			return todolist.filter(function (todo) {
				return todo.checked;
			});
		}
  },
  computed:{
    // filterTodos: function () {
		// 		return this.filters[this.filterText](this.todolist);
		// 	},
    remainingCount() {
				return this.todolist.filter(todo => !todo.checked).length
    },
    
    // filterTodos() {
		// 		switch (this.filterText) {
		// 			case 'active':
		// 				return this.todolist.filter(t => !t.checked);
		// 				break
		// 			case 'completed':
		// 				return this.todolist.filter(t => t.checked);
		// 				break
		// 			default:
		// 				return this.todolist;
		// 		}
 
    // },
    allDone: {
				get: function () {
					return this.remainingCount === 0;
				},
				set: function (value) {
					this.todolist.forEach(function (todo) {
						todo.checked = value;
					});
				}
		}
    

   },
  watch: {
			todolist: {
				handler(val, oldVal) {
					window.localStorage.setItem('todolist', JSON.stringify(val))
				},
				deep: true,
      }
  },
  
  methods:{
      
  	  addTask:function(){
        if(this.inputTask == ''){
            return false
        }
        this.todolist.push({'text':this.inputTask,checked:false,isEditing: false});
        console.log(this.todolist);
        this.inputTask = ''
      },
      checkedOne:function(index){
             console.log(this.todolist[index].checked)
      },
      deleOne:function(index){
         this.todolist.splice(index,1)
      },
      toEdit:function(todo) { 
        todo.isEditing = true;
        this.beforeEditCache = todo.text;
				// this.editedTodo = todo;
      },
      
      unEdit:function(todo,index) { 
          todo.isEditing = false
          // if (!this.editedTodo) {
          //   return;
          // }
          // this.editedTodo = null;
          // todo.text = todo.text.trim();
          if (!todo.text) {
            this.deleOne(index);
          }
      },
      cancelEdit: function (todo) {
				// this.editedTodo = null;
        todo.text = this.beforeEditCache;
         todo.isEditing = false
			},
      clearDone:function(){
          this.todolist=this.todolist.filter((item,index)=>{
              return !item.checked;
          });

      },
      changeState:function(state){
          this.state=state
      },
      displayByState: function(todo) {
        if (this.state==0) {
          return true;
        } else if (this.state==1) {
          return todo.checked==false;
        } else if (this.state==2) {
          return todo.checked==true;
        }
      }
      
  },
  directives: { 
    "todo-focus": function (el, binding) {
        if (binding.value) {
            el.focus()
        }
    }
  }
}

</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}

h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}
.container{margin-left: 45px}
.taskList{margin-left: 45px}
.text{font-size: 15px;display: inline-block;width: 200px;}
.check{text-decoration: line-through;display: block;color: #888888; }
.check .text{text-decoration: line-through;}
.flex{display: flex;justify-content: space-around;width: 50%;}
.edit{font-size: 15px;display: inline-block;width: 200px;margin: 15px 0}
.filters a{color: #353535;}
.filters a.selected{
	color:rgb(175, 47, 47);;
}
</style>
