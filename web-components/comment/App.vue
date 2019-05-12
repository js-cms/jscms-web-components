<template>
  <div id="app">
    <div class="doc_item">
      <p>标题</p>
      <p>段落2</p>
    </div>
    <!--<v-comment-items></v-comment-items>-->
    <v-my-comment></v-my-comment>
  </div>
</template>

<script>
  import CommentItems from './mycomponents/CommentList.vue'
  import MyComment from './mycomponents/MyComment.vue'

  export default {
    components: {
      'v-comment-items':CommentItems,
      'v-my-comment':MyComment
    },
    data(){
      return{
        comments:{
          userAndDocInfo: {
            "userId": "1",   // 当前用户id
            "userName": "user123", // 当前用户名称
            "userImg":"./assets/logo.png", // 当前用户头像
            "docId": "123"    // 当前文档id
          }
          ,
          items: [{
            "commentId":"123", // 评论ID
            "docId": "123", // 文档ID
            "userId": "1",  // 用户ID
            "userName": "user123",  // 用户名称
            "userImg": "/assets/logo.png",  // 用户头像
            "date": "2019-5-1",     // 时间
            "commentContent": "写得真不错",  // 评论内容
            "myApplaud": false,    // 当前用户是否赞
            "applaud": 5,           // 总赞个数
            "deriveList": [{        // 第二级以后的评论
              "commentId":"123", // 评论ID
              "userId": "2",            // 用户Id
              "userName": "user456",    // 用户名称
              "userImg": "/assets/logo.png",  // 用户头像
              "date": "2019-5-2",    // 评论发生时间，此次接口建议使用时间戳
              "commentContent": "我觉得写的非常好",  // 评论内容
              "applaud": 3,           // 表示赞的个数
              "myApplaud": false,     // 表示本账号是否赞当前评论
              "cl": 5,                 // 来表示回复层级深度
              "replyUserId": "3",       // 第二级回复的用户Id  (user456 回复的谁）
              "replyUserName": "user798"  // 第二级回复的用户名称
            }, {
              "commentId":"123", // 评论ID
              "userId": "3",
              "userName": "user789",
              "userImg": "/assets/logo.png",
              "date": "2019-5-3",
              "commentContent": "我觉得写的非常好+1",
              "applaud": 1,
              "myApplaud": false
            }
            ]
          }
          ]
        }
      }
    },
    methods:{
      addOneComment(item){// 添加评论的方法
        // item:
        // - docId   :文档ID             不可为空
        // - userId  :当前用户ID          不可为空
        // - cl      :当前评论深度  数字型  不可为空
        // - replyUserId: 回复的用户ID  可为空
        // - commentId :回复的评论ID    可为空
        console.log("添加了评论")
        return 123      // 返回新增的评论ID
      },
      changeApplaud(applaud){
        // applaud :
        // - docId :文档ID      不可为空
        // - userId :当前用户ID  不可为空
        // - commentId : 评论ID  不可为空
        // - state : 状态        不可为空
        console.log("改变了赞")
      }
    }
  }
</script>


<style>


  .iconfont {
    font-family: "iconfont" !important;
    font-size: 16px;
    font-style: normal;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }

</style>
