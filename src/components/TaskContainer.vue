<template>
    <article class="task_container">
    
      <button type="button" class="btn_empty" @click="taskClear">清空</button>
    
      <h1 class="title1">待辦事項</h1>
      <!-- input.task_name元素在 focus及 blur事件觸發後，分別替 div.task_add_block元素加 -on這個 class 及移除 -on這個 class -->
      <div class="task_add_block" ref="taskAddBlock">
        <input
        type="text"
        class="task_name" 
        placeholder="輸入待辦事項…"
        @focus="toggleShadow"
        @blur="toggleShadow"
        v-model.trim="taskText"
        @keyup.enter="taskAdd">
        <!-- v-model 雙向綁定 -->

        <button 
        type="button" 
        class="task_add"
        @click="taskAdd">新增</button>
      </div>
    
      <div class="task_list_parent">
    
        <ul class="task_list" ref="taskList">

          <!-- 元件標籤 -->
          <TaskItem 
          :tasks="tasks" 
          @task-edit="taskEdit"
          @task-star="taskStar"
          @task-swap="taskSwap"
          @task-remove="taskRemove"></TaskItem>

        </ul>
    
      </div>
    </article>
  </template>

<script>
// 區域掛載
    import TaskItem from "./TaskItem.vue";
  
    export default {
      components:{TaskItem},
        data(){
            return{
              taskText:"",
              tasks:[
                // {
                //   id:"a",
                //   name:"一",
                //   star:0,
                //   editable:false
                // }
              ]
            };
        },
        beforeMount(){
          // console.log("ttt");
          let tasks = JSON.parse(localStorage.getItem("tasks"));
          // console.log(tasks);
          if(tasks){
            this.tasks = tasks;
          }
          
        },
        methods:{
            toggleShadow(){
                // console.log("aaaaa");
                // console.log(this.$refs.taskAddBlock);
                this.$refs.taskAddBlock.classList.toggle("-on");
            },
            taskAdd(){
                if( this.taskText != ""){
                    // alert("yyy");
                    //把資料放在陣列最前面，使用unshift
                    this.tasks.unshift({
                      id:Date.now(),
                      name:this.taskText,
                      star:0,
                      editable:false
                    });
                    this.taskText="";

                    localStorage.setItem("tasks" , JSON.stringify(this.tasks));
                }
            },
            taskClear(){
              let r = confirm("確認要清空嗎？")
              if(r){
                // console.log(this.$refs.taskList);
                
                for(let i = 0 ; i < this.$refs.taskList.children.length; i++){
                   console.log(i);
                   this.$refs.taskList.children[i].classList.add("fade_out");   
                }

                setTimeout(()=>{
                  // console.log(this);
                  this.tasks =[];
                  localStorage.clear();
                },1000);

              }
            },
            taskEdit(e,i){
              // console.log(e);
              //console.log(i);
             if(this.tasks[i].editable){
              if(this.tasks[i].name ==""){
                alert("請輸入資料")
              }else{
                this.tasks[i].editable = !this.tasks[i].editable;
                localStorage.setItem("tasks" , JSON.stringify(this.tasks));
              }
             }else{
               this.tasks[i].editable = !this.tasks[i].editable;
             }
            },
            taskStar(e,i,star){
            // alert(star);
            this.tasks[i].star = star;
            localStorage.setItem("tasks" , JSON.stringify(this.tasks));
            },
            taskSwap(e, i, direction){
              // alert(direction);
              //用 direction 做出往上往下的區分，先想辦法排除灰色按鈕(用索引質判斷)，如何往上往，使用兩個索引質對調
              if( direction == "up" && i != 0){
                // alert("up");
                [this.tasks[i-1] , this.tasks[i]] = [this.tasks[i] , this.tasks[i-1]];
                localStorage.setItem("tasks" , JSON.stringify(this.tasks));
              }

              if(direction == "down" && i != this.tasks.length - 1){
                // alert("down");
                [this.tasks[i] , this.tasks[i+1]] = [this.tasks[i+1] , this.tasks[i]];
                localStorage.setItem("tasks" , JSON.stringify(this.tasks));

              }
            },
            taskRemove(e , i){
              // console.log(i);
              let r = confirm('確認要移除嗎？')
              if(r){
                e.target.closest("li").classList.add("fade_out");

                setTimeout(()=>{
                  // 從索引值開始刪掉1個資料
                  this.tasks.splice(i , 1);
                  localStorage.setItem("tasks" , JSON.stringify(this.tasks));
                },1000)
              }
            }
        }
    }
</script>

<style lang="sass" scoped>
  article.task_container
    width: 600px
    margin: 0 auto
    border-radius: 5px
    padding: 10px
    box-shadow: 1px 1px 5px gray
    max-width: 100%
    h1.title1
      font-size: 2.4rem
      margin: 0 0 10px 0
  
    /* ===== 任務新增 ===== */
    div.task_add_block
      font-size: 0
      transition: transform .5s, box-shadow .5s
      &.-on
        box-shadow: 0 0 5px gray
        transform: scale(1.01)
        transform-origin: center center
  
      input.task_name
        width: calc(100% - 50px)
        border: 1px solid lightgray
        border-radius: 3px 0 0 3px
        height: 40px
        font-size: 2rem
        padding: 5px 10px
        outline: none
        display: inline-block
        vertical-align: top
    input.task_name::placeholder
      color: lightgray
  
    div
      &.task_add_block button.task_add
        display: inline-block
        width: 50px
        height: 40px
        vertical-align: top
        border: 1px solid lightgray
        border-left: 0
        background-color: white
        box-shadow: none
        font-size: 1.6rem
        cursor: pointer
        outline: none
        border-radius: 0 3px 3px 0
  
        &:active
          box-shadow: 1px 1px 3px lightgray inset, -1px -1px 3px lightgray inset
          background-color: #fcfcfc
          font-size: 1.5rem
      &.task_list_parent
        margin-top: 30px
        ul.task_list
          margin: 0
          padding: 0
          list-style: none
  
    /* ===== 清空按鈕 ===== */
    button.btn_empty
      float: right
      background: none
      background-color: white
      padding: 0
      margin: 0
      border: 1px solid lightgray
      border-radius: 3px
      height: 30px
      width: 50px
      outline: none
      font-size: 1.6rem
      cursor: pointer
</style>