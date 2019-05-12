<template>
  <div class="comment-items-parent">
    <div class="comment-part">
      <div class="comment-part-title">
        <div class="my-column-title"><span class="my-column-title-icon"></span><span
          class="my-column-title-text weight-bold">参与评论</span></div>
      </div>
      <div class="comment-input-content">
        <div class="comment-input-wrapper">
          <textarea id="js_comment_input" class="comment-input" @input="descArea"
                    v-model="userCommentItem.commentContent"
                    placeholder="你怎么看...">

          </textarea>
        </div>
        <div class="comment-control" style="height: 30px;">
          <a class="comment-control-user">
            <span class="user-avatar" style=""><img v-bind:src="userCommentItem.userImg" /> </span>
            {{userCommentItem.userName}}
          </a>
          <span class="comment-control-user-reply" v-if="userCommentItem.replyUserId">
          回复: {{userCommentItem.replyUserName}} <span class="iconfont reply-close"
                                                      @click="clearReplyUser">&#xe7fc; </span>
        </span>
          <span
            class="comment-control-submit" @click="commit">
          <span>提交评论</span>
        </span>
          <span class="length-tip">{{wordLengths}}/1000</span>
        </div>
      </div>
      <p class="comment-err-tip" v-if="errorInput">输点东西后再提交哦 ^_^</p>
      <p class="comment-input-warn">请回复有价值的信息，无意义的评论将很快被删除，账号将被禁止发言。</p>
    </div>

    <div class="comment-part">
      <div class="comment-part-title">
        <div class="my-column-title">
          <span class="my-column-title-icon"></span>
          <span class="my-column-title-text weight-bold">评论区</span>
        </div>
      </div>
      <ul class="comment-list">
        <li class="comment-list-item" v-for="(item,i) in items ">
          <div class="item-left">
            <span class="item-avatar" style=""> <img v-bind:src="item.userImg"/>	</span>
          </div>
          <div class="item-right">
            <div class="item-user">
              <span class="item-user-name">{{ item.userName }}</span>
              <span class="item-user-time"> · {{ item.date }}</span>
              <div class="item-user-right">
                <span class="item-user-like" v-bind:class="{'item-user-unlike':!item.myApplaud}"
                      @click="changeApplaud([i],$event)"> <span
                  class="iconfont">&#xe7c8;</span> {{item.applaud}}</span>
                <a class="item-user-reply" @click="click_repty([i],$event)">回复</a>
              </div>
            </div>
            <p class="item-text">{{ item.commentContent }}</p>
            <div class="comment-list comment-list-reply" v-for=" (citem,ci) in item.deriveList">
              <ul class="comment-list-ul">
                <li class="comment-list-item">
                  <div class="item-right">
                    <div class="item-user">
                      <span class="item-user-name">{{ citem.userName}}</span>
                      <span class="item-user-time"> · {{citem.date}}</span>
                      <div class="item-user-right">
                        <span class="item-user-like " v-bind:class="{'item-user-unlike':!citem.myApplaud}"
                              @click="changeApplaud([i,ci],$event)"> <span class="iconfont">&#xe7c8;</span> {{citem.applaud}}</span>
                        <a class="item-user-reply" @click="click_repty([i,ci],$event)">回复</a>
                      </div>
                    </div>
                    <p class="item-text">
                      <span class="item-user-reply" v-if="citem.cl>3">
                        回复 {{citem.replyUserName}} :
                      </span>{{ citem.commentContent }}</p>
                  </div>
                </li>
              </ul>
            </div>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>

  export default {
    name: "MyComment",
    data() {

      let cd = {};
      Object.assign(cd,this.$parent.comments);

      return {
        userCommentItem: {
          "docId": cd.userAndDocInfo.docId,
          "userId": cd.userAndDocInfo.userId,
          "userName": cd.userAndDocInfo.userName,
          "userImg":cd.userAndDocInfo.userImg,
          "commentContent": "",
          "replyUserId": "",
          "replyUserName": "",
          "replyPath": [],
          "cl": 0,
          "applaud": 0,           // 表示赞的个数
          "myApplaud": false,
          "commentId":0
        }
        ,
        items: cd.items,
        wordLengths: 0,
        wordMax: 1000,
        errorInput: false
      }
    }
    ,
    methods: {
      click_repty(e) {
        this.userCommentItem.replyPath = e;

        if (e.length == 1) {
          // 第一层的回复
          this.userCommentItem.replyUserId = this.items[e[0]].userId;
          this.userCommentItem.replyUserName = this.items[e[0]].userName
          this.userCommentItem.cl=(this.items[e[0]].cl||0) +1
        } else {
          // 第二层的回复
          this.userCommentItem.replyUserId = this.items[e[0]].deriveList[e[1]].userId;
          this.userCommentItem.replyUserName = this.items[e[0]].deriveList[e[1]].userName
          this.userCommentItem.cl=(this.items[e[0]].deriveList[e[1]].cl||0) +1
        }
      }
      ,
      commentAdd(userCommentItem) {
        // 直接文档的评论
        // 对第一层的评论
        // 通过参数可以找到replyPath 可以定位到对谁的评论
        // 多层评论
        // 通过参数可以找到replyPath 可以定位到对谁的评论
        userCommentItem.date = new Date();
        userCommentItem.applaud = 0;
        // 此处调用接口将数据存入数据库,接收返回的ID
        userCommentItem.commentId = this.$parent.addOneComment(userCommentItem);

        if (userCommentItem.replyPath.length == 0) {
          this.items.push(userCommentItem);
        } else if (userCommentItem.replyPath.length == 1) {
          if (!this.items[userCommentItem.replyPath[0]].deriveList) {
            this.items[userCommentItem.replyPath[0]].deriveList = []
          }
          this.items[userCommentItem.replyPath[0]].deriveList.push(userCommentItem)

        } else if (userCommentItem.replyPath.length == 2) {
          this.items[userCommentItem.replyPath[0]].deriveList.push(userCommentItem)
        }


        this.userCommentItem.commentContent="";
        this.userCommentItem.replyUserId="";
        this.userCommentItem.replyUserName="";
        this.userCommentItem.replyPath=[];
        this.userCommentItem.cl=0;
        this.userCommentItem.applaud=0;
        this.userCommentItem.myApplaud=false;
      }
      ,
      changeApplaud(e) {  // 点赞
        this.userCommentItem.replyPath = e;

        let applaud={};
        applaud.docId = this.userCommentItem.docId;
        applaud.userId = this.userCommentItem.userId;

        if (e.length == 1) {
          // 第一层的赞
          this.$set(this.items[e[0]], 'myApplaud', !this.items[e[0]].myApplaud);
          this.items[e[0]].applaud = this.items[e[0]].applaud + (this.items[e[0]].myApplaud || -1);
          applaud.commentId=this.items[e[0]].commentId;
          applaud.state=!this.items[e[0]].myApplaud;

        } else {
          // 第二层的赞
          this.$set(this.items[e[0]].deriveList[e[1]],'myApplaud',!this.items[e[0]].deriveList[e[1]].myApplaud)
          this.items[e[0]].deriveList[e[1]].applaud = this.items[e[0]].deriveList[e[1]].applaud + (this.items[e[0]].deriveList[e[1]].myApplaud || -1)

          applaud.commentId=this.items[e[0]].deriveList[e[1]].commentId;
          applaud.state=!this.items[e[0]].deriveList[e[1]].myApplaud;
        }

        this.$parent.changeApplaud(applaud)

        this.userCommentItem.replyPath=[]
      }
      ,
      clearRu() {   //清除输入框的回复人信息
        this.userCommentItem.replyUserId = "";
        this.userCommentItem.replyUserName = "";
        this.userCommentItem.replyPath = [];
        this.userCommentItem.cl=0;
      },
      commit() { // 提交回复
        if (this.userCommentItem.commentContent.length < 1) {
          this.errorInput = true;
          return;
        }
        let myItem = {};
        Object.assign(myItem,this.userCommentItem);
        this.commentAdd(myItem);
        this.wordLengths = 0;
      },
      descArea() {  // 用于字数监听
        this.wordLengths = this.userCommentItem.commentContent.length;
        if (this.wordLengths > 0) {
          this.errorInput = false;
        }
        if (this.wordLengths > this.wordMax) {
          this.userCommentItem.commentContent = this.userCommentItem.commentContent.substr(0, this.wordMax);
        }
      },
      clearReplyUser() {
        this.clearRu();
      }
    }
  }
</script>

<style scoped>

  .comment-items-parent {
    font-size: 12px;
    text-align: left;
    position: absolute;
    display: block;
    width: 95%;
    margin: 0;
  }

  .comment-part {
    font-size: 14px;
    width: 100%;
  }

  .comment-part-title {
    margin-top: 40px;
    padding-bottom: 3px;
  }

  .my-column-title {
    height: 28px;
    line-height: 28px;
  }

  .my-column-title-icon {
    /*background: url("./assets/logo.png") no-repeat;*/
    display: inline-block;
    padding: 7px;
  }

  .my-column-title-text {
    color: #262626;
    font-size: 20px;
    padding-left: 4px;
  }

  ol, ul {
    list-style: none;
  }

  ul li {
    display: block;
  }

  .item-left {
    left: -50px;
    position: absolute;
    top: 17px;
    width: 40px;
  }

  .comment-list-item .item-right {
    border-bottom: 1px solid #ddd;
    padding: 0;
    margin: 0;
  }

  .item-right {
    color: #252525;
    padding: 0px;
  }

  .item-user {
    font-size: 12px;
    line-height: 20px;
    margin: 10px 0;
  }

  .item-user-name {
    font-size: 14px;
  }

  .item-user-time {
    color: #a7a7a7;
  }

  .item-user-right {
    float: right;
  }

  .item-user-like.item-user-unlike {
    color: #787878;
  }

  .item-user-like {
    /*background: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADAAAAAwCAYAAABXAvmHAAAAAXNSR…P4XzYAE+BD5BStpuM85n340Y9mQDOgGdAMaAY0A/8lAz8Bc5fA/BF5WqMAAAAASUVORK5CYII"/assets/logo.png") no-repeat 0;*/
    background-size: 22px;
    color: #f95355;
    cursor: pointer;
    display: inline;
    margin-right: 10px;
    padding-left: 24px;
  }

  .item-user-reply {
    color: #4285f4;
    cursor: pointer;
  }

  .item-text {
    white-space: pre-line;
    padding: 0;
  }

  .item-text .item-user-reply {
    color: #787878;
    display: inline-block;
  }

  .comment-list-reply {
    background-color: #edf1f4;
    border-radius: 2px;
    margin-top: 15px;
    position: relative;
  }

  .comment-list-ul {
    /*padding:0 15px;*/
  }

  .item-right .comment-list-ul .comment-list-item {
    /*border-bottom: 1px dotted #ddd;*/
    margin-left: 0;
    margin-right: 0;
  }

  .comment-list .comment-list-item {
    /*border-bottom: 1px solid #ddd;*/
    margin-left: 50px;
    padding: 0;
    position: relative;
  }

  comment-list .comment-list-item .item-right {
    padding: 0px;
  }

  textarea {
    outline: none;
  }

  .my-column-title-text {
    color: #262626;
    font-size: 20px;
    padding-left: 4px;
  }

  .comment-part-title {
    margin-top: 40px;
    padding-bottom: 3px;
  }

  .comment-input-content {
    text-align: left;
    background: rgba(245, 247, 249, .8);
    border: 1px solid #ddd;
    border-radius: 2px;
  }

  .comment-input-wrapper {
    display: flex;
  }

  .comment-input-content .comment-input-wrapper .comment-input {
    border: 0;
    color: #333;
    flex: 1;
    font-size: 14px;
    height: 110px;
    padding: 10px;
    resize: none;
  }

  .comment-input-content .comment-control {
    /*height: 50px;*/
    line-height: 30px;
    padding: 10px;
  }

  .comment-input-content .comment-control .comment-control-user {
    color: #4285f4;
    float: left;
    height: 30px;
    margin-right: 30px;
    padding-left: 40px;
    position: relative;
  }

  .comment-input-content .comment-controller-user .user-avatar {
    background: no-repeat 50% #787878;
    background-size: cover;
    border-radius: 50%;
    height: 30px;
    left: 0;
    position: absolute;
    top: 0;
    width: 30px;
  }

  .comment-input-content .comment-control-submit {
    -moz-user-select: none;
    -ms-user-select: none;
    -webkit-user-select: none;
    background-color: #4285f4;
    border-radius: 1px;
    color: #fff;
    cursor: pointer;
    float: right;
    height: 30px;
    padding: 0 12px;
    position: relative;
    user-select: none;
  }

  .comment-input-content .length-tip {
    color: #787878;
    float: right;
    font-size: 14px;
    padding-right: 10px;
  }

  .comment-control-user-reply {
    background-color: #e1e9f6;
    border-radius: 2px;
    color: #4285f4;
    float: left;
    height: 30px;
    padding: 0 60px 0 10px;
    position: relative;
    display: block;
  }

  .reply-close {
    text-align: center;
    background-size: 14px;
    cursor: pointer;
    height: 30px;
    position: absolute;
    right: 0;
    top: 0;
    transition: all .5s;
    width: 30px;
  }

  .reply-close:hover {
    -webkit-transform: rotate(180deg);
    transform: rotate(180deg);
  }

  .comment-err-tip {
    color: red;
  }


</style>
