<template>
<!-- Content Header (Page header) -->
<div class="content-header">
    <div class="container-fluid">
        <div class="row mb-2">
            <div class="col-sm-6">
                <h1 class="m-0" v-if="this.category == 'TALK'">Post Detail Talk</h1>
                <h1 class="m-0" v-else-if="this.category == 'QA'">Post Detail Q&A</h1>
            </div>
            <!-- /.col -->
           
            <!-- /.col -->
        </div>
        <!-- /.row -->
    </div>
    <!-- /.container-fluid -->
</div>
<!-- /.content-header -->

<!-- Main content -->
<section class="content">
    <div class="container-fluid">
        <div>
                       <!-- ////////////////////////////////////////////////// -->
                        <div class="card card-primary card-outline">
                        <div class="card-header">
                        <h3 class="card-title"><i class="nav-icon fas fa-user"></i> &nbsp;{{writerNickname}}<br><i class="nav-icon fas fa-edit"></i>&nbsp;{{modifiedDate}}</h3>
                        <div style="float:right">
                        <i class="fas fa-eye fa-fw" style="color: var(--fa-navy);"></i>&nbsp;{{hit}} <br> <i class="fas fa-comment-dots fa-fw" style="color: var(--fa-navy);"></i>&nbsp;{{comment_count}}
                        </div>
                        </div>

                        <div class="card-body">
                        <div class="form-group">
                                <select class="form-control" v-model="category" disabled> 
                                      <option value="TALK">Talk</option>
                                      <option value="QA">Q&A</option> 
                                  </select>
                        </div>
                        <div class="form-group">
                        <input class="form-control" placeholder="Title" v-model="title" disabled>
                        </div>
                        <div class="form-group"> 
                        <div class="col-md" style="padding:0px">
                        <div class="card card-info">
                        <div class="card-header" style="height:40px">
                        <span class="card-title"></span>
                        <div class="card-tools">
                        </div>
                        </div>
                        <div class="card-body" style="display: block;">
                                <p v-html="content" style="min-height:200px"></p>
                        </div>
                        </div>
                        </div>
                        </div>
                        </div>
                        <div class="card-footer">
                        <div class="float-right">
                        <button v-if="roles == 'ROLE_ADMIN'" type="submit" class="btn btn-warning" @click="setUpdate">C</button>    
                        <button v-if="userid == writerUsername || roles == 'ROLE_ADMIN'" type="submit" class="btn btn-danger" style="margin-left: 15px;" @click="setDelete">Delete</button>
                        <button v-if="userid == writerUsername || roles == 'ROLE_ADMIN'" type="submit" class="btn btn-primary" style="margin-left: 15px;" @click="goEdit">Edit</button>
                        <button type="submit" class="btn btn-info" style="margin-left: 15px;" @click="goPost">List</button>
                        </div>
                        </div>

                        </div>
                        <!-- ////////////////////////////////////////////////// -->
                            <div class="col-12"  :key="index" v-for="(comment,index) in postComments">
                                <div class="card">
                                <div class="card-header">
                                <h3 class="card-title"><i class="nav-icon fas fa-user"></i> &nbsp;{{comment.nickname}}<br/><i class="nav-icon fas fa-edit"></i>&nbsp;{{comment.modifiedDate}}</h3>
                                <div class="card-tools">
                                <button  v-if="userid == comment.username || roles == 'ROLE_ADMIN'"  type="button" class="btn btn-tool" data-card-widget="remove" title="Remove" @click="deleteComment(comment.postCommentId)">
                                <i class="fas fa-times"></i>
                                </button>
                                </div>
                                </div>
                                <div class="card-body">
                                    <div style="white-space:pre-wrap;word-break:break-all">{{comment.content}}</div>
                                </div>
                                </div>
                            </div>
                    <!-- ////////////////////////////////////////////////// -->

                    <div v-if="currentUser" class="card card-success" style="margin:15px">
                            <div class="card-header">
                            <h3 class="card-title">Reply</h3>
                            </div>
                            <div class="card-body">
                                    <textarea rows="5" class="form-control"  placeholder="Comment를 남겨 보세요." maxlength="1000" v-model="comment" ref="comment"></textarea>
                                    <br>
                                    <div style="float:right"><button  type="submit" class="btn btn-danger" style="margin-left: 15px;" @click="setComment">Submit</button></div>
                            </div>
                    </div>


                    <div v-else-if="!currentUser" class="callout callout-info" style="margin:10px">
                    <h5><i class="fas fa-info"></i> Notice</h5>
                        <p style="text-align:center">로그인을 하시면 댓글 작성이 가능합니다.</p>
                    </div>


        </div>
    </div>
    <!-- /.container-fluid -->
</section>


<!-- /.content -->
</template>

<script>
import PostService from "../services/post.service";

export default {
  name: "postdetail",
        data() {
            return {
                category : 'TALK',
                title : '',
                content : '',
                hit : 0,
                comment_count: 0,
                comment : '' , 
                writerNickname : "",
                writerUsername : "",
                modifiedDate : "",
                postComments : [],
            };
        },
        created() {
           PostService.getPost(this.$route.query.postId).then(
                    (response) => {
                        if(response.data.result == 'S'){
                            this.category = response.data.data.category;
                            this.title = response.data.data.title;
                            this.content = response.data.data.content;
                            this.hit = response.data.data.hit;
                            this.writerNickname = response.data.data.nickname;
                            this.writerUsername = response.data.data.username;
                            this.modifiedDate = response.data.data.modifiedDate;
                        }else{
                            this.$toast.error(`Fail.`);
                        }
                    },
                    (error) => {
                        this.$toast.error(`Fail.`);
                        console.log(
                        (error.response &&
                            error.response.data &&
                            error.response.data.message) ||
                        error.message ||
                        error.toString());
                    }
            );

            this.getPostCommentList();
        },
        computed: {
            currentUser() {
                return this.$store.state.auth.user;
            }
        },
        mounted(){
            if(this.currentUser){
                const user = JSON.parse(localStorage.getItem('user'));
                 this.userid = user.username;
                 this.nickname = user.nickname;
                 this.email = user.email;
                 this.roles= user.roles[0];
            }
        },
          methods: {
            goPost() {
                sessionStorage.setItem('post_back', 'Y');
                this.$router.push({
                    name: "Post",
                    query: { category: this.category },
                });
            },
            goEdit() {
                this.$router.push({
                    name: "PostEdit",
                    query: { category: this.category, postId: this.$route.query.postId },
                });
            },
             setDelete() {
                    this.$confirm("삭제 하시겠습니까?").then(() => {
                            PostService.deletePost(this.$route.query.postId).then(
                                (response) => {
                                    if(response.data.result == 'S'){
                                        this.$toast.success(`Success.`);
                                        this.$router.push({
                                            name: "Post",
                                            query: { category: this.category },
                                        });
                                    }else{
                                            this.$toast.error(`Fail.`);
                                    }
                                },
                                (error) => {
                                    this.$toast.error(`Fail.`);
                                    console.log(
                                    (error.response &&
                                        error.response.data &&
                                        error.response.data.message) ||
                                    error.message ||
                                    error.toString());
                                }
                        );
                    
                  }).catch((e) => e !== undefined ?  this.$toast.error(`Fail. ->`+e) : console.log('no selected =>'+e));
            },
             setUpdate() {
                    this.$confirm("Category를 변경 하시겠습니까?").then(() => {
                            PostService.changePostCategory(this.$route.query.postId).then(
                                (response) => {
                                    if(response.data.result == 'S'){
                                        this.$toast.success(`Success.`);
                                        this.$router.push({
                                            name: "Post",
                                            query: { category: response.data.data },
                                        });
                                    }else{
                                            this.$toast.error(`Fail.`);
                                    }
                                },
                                (error) => {
                                    this.$toast.error(`Fail.`);
                                    console.log(
                                    (error.response &&
                                        error.response.data &&
                                        error.response.data.message) ||
                                    error.message ||
                                    error.toString());
                                }
                        );
                    
                  }).catch((e) => e !== undefined ?  this.$toast.error(`Fail. ->`+e) : console.log('no selected =>'+e));
            },
            getPostCommentList(){
                PostService.getPostCommentList(this.$route.query.postId).then(
                    (response) => {
                        if(response.data.result == 'S'){
                             this.postComments = response.data.data; 
                             this.comment_count= this.postComments.length;
                        }else{
                             this.postComments = [];
                             this.comment_count= 0;
                             this.$toast.error(`Fail.`);
                        }
                    },
                    (error) => {
                         this.comment_count= 0;
                        this.postComments = [] ;
                        this.$toast.error(`Fail.`);
                        console.log(
                        (error.response &&
                            error.response.data &&
                            error.response.data.message) ||
                        error.message ||
                        error.toString());
                    }
               );
            },
            deleteComment(postCommentId) {
                    this.$confirm("삭제 하시겠습니까?").then(() => {
                            PostService.deletePostComment(postCommentId).then(
                                (response) => {
                                    if(response.data.result == 'S'){
                                        this.getPostCommentList();
                                        this.$toast.success(`Success.`);
                                    }else{
                                        this.$toast.error(`Fail.`);
                                    }
                                },
                                (error) => {
                                    this.$toast.error(`Fail.`);
                                    console.log(
                                    (error.response &&
                                        error.response.data &&
                                        error.response.data.message) ||
                                    error.message ||
                                    error.toString());
                                }
                        );
                    
                 }).catch((e) => e !== undefined ?  this.$toast.error(`Fail. ->`+e) : console.log('no selected =>'+e));
            },
            setComment() {
                 if( this.comment.trim() == ''){
                    this.$toast.warning(`Comment 내용을 입력해 주세요.`);
                    this.$refs.comment.focus();
                    return;
                }

                    this.$confirm("저장 하시겠습니까?").then(() => {
                            PostService.setPostComment({postId : this.$route.query.postId, content : this.comment}).then(
                                (response) => {
                                    this.comment = '';
                                    if(response.data.result == 'S'){
                                        this.getPostCommentList();
                                        this.$toast.success(`Success.`);
                                    }else{
                                        this.$toast.error(`Fail.`);
                                    }
                                },
                                (error) => {
                                    this.comment = '';
                                    this.$toast.error(`Fail.`);
                                    console.log(
                                    (error.response &&
                                        error.response.data &&
                                        error.response.data.message) ||
                                    error.message ||
                                    error.toString());
                                }
                        );
                    
                  }).catch((e) => e !== undefined ?  this.$toast.error(`Fail. ->`+e) : console.log('no selected =>'+e));
            },
        },
};
</script>
