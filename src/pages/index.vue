<template lang="html">
<div id="container">
<cvHead></cvHead>
<main id="main">
    <el-row :gutter="20" id="container">
        <el-col :span="18" id="content" :offset="3">
            <div class="grid-content bg-purple">
                <el-card class="box-card index">
                    <div slot="header" class="clearfix index-nav">
                        <el-menu id="navbar" theme="light" :default-active="curTab" class="el-menu-demo" mode="horizontal" router>
                            <el-menu-item index="all" :route="{name: 'index', query: {tab: 'all'}}">全部</el-menu-item>
                            <el-menu-item index="good" :route="{name: 'index', query: {tab: 'good'}}">精华</el-menu-item>
                            <el-menu-item index="share" :route="{name: 'index', query: {tab: 'share'}}">分享</el-menu-item>
                            <el-menu-item index="ask" :route="{name: 'index', query: {tab: 'ask'}}">问答</el-menu-item>
                            <el-menu-item index="job" :route="{name: 'index', query: {tab: 'job'}}">招聘</el-menu-item>
                        </el-menu>
                    </div>
                    <cvList :topics="topics"  v-if="topics.length"></cvList>
                    <section class="page">
                        <!-- <cvPage></cvPage> -->
                    </section>
                </el-card>
            </div>
        </el-col>
    </el-row>
</main>
<cvLoading :showLoading="loading.showLoading"></cvLoading>
</div>

</template>

<script>
import cvHead from "../components/header.vue";
import cvLoading from "../components/loading.vue";
import cvList from  "../components/list.vue";

export default {
    data() {
        return {
            topics: [],
            curTab: this.$route.query.tab || "all",
            queryData: {
                page: 1,
                tab: "all",
                limit: 20,
                mdrender: true
            },
            loading: {
                showLoading: false,
                show() {
                    this.showLoading = true;
                },
                hide() {
                    this.showLoading = false;
                }
            },
            scrollLock: false
        };
    },
    computed: {},
    watch: {
        "$route" () {
            this.curTab = this.$route.query.tab || "all";
            this.fetchTopics();
        }
    },
    mounted() {
        //从别的页进入主页，且tab相同，则加载缓存数据
        //这里我在想如果sessionStorage里有curTab，要不要点击首页时重定向到curTab？
        //待定待定
        if(sessionStorage.queryData && sessionStorage.curTab === this.curTab){
            this.topics = JSON.parse(sessionStorage.topics || "[]");
            this.queryData = JSON.parse(sessionStorage.queryData || this.queryData);
            let scrollTop = sessionStorage.scrollTop || 0;
            this.$nextTick(() => {
                $(window).scrollTop(scrollTop)
            });
        }else{
            this.fetchTopics();
        }
        //上拉加载
        $(window).on("scroll", (e) => {
            this.scrollLoad();
        });
        sessionStorage.removeItem("curTab");
        sessionStorage.removeItem("topics");
        sessionStorage.removeItem("queryData");
        sessionStorage.removeItem("scrollTop");

    },
    beforeRouteLeave (to, from, next) {
        //如果从首页进入用户详情和话题详情页面的话，将当前话题，滚动条高度，当前queryData缓存进sessionStorage
        //因为其他例如api,about,newtopic页面都是在顶部才能进去
        //后续如果header fixed的话，可以把限制条件去掉
        //
        //Bug: 点击去往详情页后，返回，此时不要滚动页面和点击链接，然后前进，此时scrollTop为0
        //待解决
        if(to.name === "user" || to.name === "topic"){
            sessionStorage.curTab = from.query.tab || "all";
            sessionStorage.topics = JSON.stringify(this.topics);
            sessionStorage.queryData = JSON.stringify(this.queryData);
            sessionStorage.scrollTop = $(window).scrollTop();
        }
        $(window).off("scroll");
        next();
    },
    methods: {
        fetchTopics() {
            this.loading.show();
            this.scrollLock = true;
            let tab = this.$route.query.tab || "all",
                tabdiff = tab === this.queryData.tab;
            this.queryData.tab = tab;
            $.ajax({
                url: "https://cnodejs.org/api/v1/topics",
                type: "GET",
                data: this.queryData
            }).done((data) => {
                this.loading.hide();
                this.scrollLock = false;
                if (!data || !data.success) {
                    //TODO 错误抛出
                    return;
                }
                data.data.forEach((v, i) => {
                    v.typeClass = this.getTypeClass(v.top, v.good, v.tab);
                });
                if(!tabdiff){
                  this.topics = [];
                }
                this.topics = this.topics.concat(data.data);

            }).fail((error) => {
                this.showLoading = false;
                //TODO 错误抛出
            });
        },
        getTypeClass(top, good, tab) {
            if (top) {
                return "success";
            } else if ( good) {
                return "danger";
            } else if (tab == "ask") {
                return "primary";
            } else if (tab == "job") {
                return "warning";
            } else if (tab == "share") {
                return "gray";
            }else if (!top && !good && !tab || (this.$route.query.tab === tab)) {
                return "hidden";
            } else {
                return "";
            }
        },
        scrollLoad: function(){
          if(this.scrollLock)
            return;
          let body = document.body,
              height = body.clientHeight + body.scrollTop,
              scrollHeight = body.scrollHeight;
          if(height > scrollHeight - 10){
            this.scrollLock = false;
            this.queryData.page++;
            this.fetchTopics();
          }
        }

    },
    components: {
        cvHead,
        cvLoading,
        cvList
    }
}


</script>

<style lang="sass">
#content {
    .index{
        .el-card__header {
            padding: 0;
            background-color: #eff2f7;
            .index-nav {}
            }
    }
}
</style>
