<template>
    <i-row type="flex" :gutter="24">
        <i-col span="15">
            <i-card :padding="24">
                <template v-slot:title>
                    <i-row type="flex" align="middle">
                        <span style="font-size: 18px;">我的待办&nbsp;</span>
                        <Badge :count="entryForManager.pending.badge" v-if="entryForManager.pending.badge > 0"></Badge>
                    </i-row>
                </template>
                <template v-slot:extra>
                   <router-link :to="{name: 'MyPending'}">查看所有待办</router-link>
                </template>
                <List v-if="pendingData.length > 0 ">
                    <ListItem v-for="(item, index) in pendingData.slice(0, 3)" :key="index">
                        <i-row type="flex" align="middle" style="width: 100%">
                            <i-col span="20">
                                <p>
                                    <Icon type="ios-checkmark-circle-outline" color="limegreen"/>
                                    {{item.WorkflowName}}
                                </p>
                                <p style="color: #808695; font-size:14px">
                                    <Icon type="md-information-circle" />
                                    {{item.Owner}}提交的流程
                                </p>
                                <p style="color: #808695; font-size:14px">
                                    <Icon type="md-shuffle" />
                                    工作流到达时间{{item.ArriveOn}}
                                </p>
                            </i-col>
                        </i-row>
                        <template v-slot:action>
                            <Button type="success" @click="dealWorkflow(item.InstanceId, item.StepId, item.WorkflowType)">审核</Button>=
                        </template>
                    </ListItem>
                </List>
                <template v-else>
                    <i-row class="picture">暂无待办事项</i-row>
                </template>
            </i-card>
            <i-card style="margin-top:10px" title="社团活动" :padding="24">
                <template v-slot:extra>
                    <i-input search placeholder="搜索活动名称" @on-search="searchActivity" />
                </template>
                <i-row>
                    <i-table stripe :columns="tableColumns.activity" :data="activityData" :loading="tableLoading">
                        <template slot="Action" slot-scope="{row}">
                            <i-button @click="checkWorkflow(row.InstanceId, row.StepId, row.ID)">查看</i-button>
                        </template>
                        <template slot="QRCode" slot-scope="{row}">
                            <img v-if="row.ShortCode" :src="'/qr/' + row.ShortCode" />
                        </template>
                    </i-table>
                    <Page :styles="{'margin-top': '16px'}" :total="pager.totalRow" show-sizer show-total :page-size="5" :page-size-opts="[5, 10, 20, 100]"
                     @on-change="getActivities($event)" @on-page-size-change="getActivities(null ,$event)" />
                </i-row>
            </i-card>
        </i-col>
        <i-col span="8">
            <i-card :padding="0" style="margin-bottom:10px">
                <div :to="dashBoard.DepartType === 0 ? routers[1]:routers[0]" v-if="true" style="background-color:#ffffff">
                    <i-row type="flex" align="middle" style="padding:16px;margin: 16px 0px">
                        <i-col span="4">
                            <Avatar :size="48" :src="userInfo.avatar"/>
                        </i-col>
                        <i-col span="19">
                            <div style="font-size: 18px; color: #17233d;">{{time}}好! {{userInfo.realName}}</div>
                            <a @click="navTo">{{orgInfo.Name}}的指导老师</a>
                        </i-col>
                    </i-row>
                    <i-row type="flex" class="background-purple">
                        <i-col span="8">
                            <div style="padding-top:20px;">
                                <i-row type="flex">
                                    <i-col span="22">
                                        <div style="font-size: 33px;text-align:center;">{{membersData.length}}</div>
                                        <div style="margin-bottom:10px;font-size:12px;text-align:center;">成员数</div>
                                    </i-col>
                                    <i-col span="1">
                                        <div style="border-right: 1px solid #fff;height: 36px;margin-top: 10px;opacity: .2;"/>
                                    </i-col>
                                </i-row>
                            </div>
                        </i-col>
                        <i-col span="8">
                            <div style="padding-top:20px;">
                                <i-row type="flex">
                                    <i-col span="22">
                                        <div style="font-size: 33px;text-align:center;">{{pager.totalRow}}</div>
                                        <div style="margin-bottom:10px;font-size:12px;text-align:center;">活动数</div>
                                    </i-col>
                                    <i-col span="1">
                                        <div style="border-right: 1px solid #fff;height: 36px;margin-top: 10px;opacity: .2;"/>
                                    </i-col>
                                </i-row>
                            </div>
                        </i-col>
                        <i-col span="8">
                            <div style="padding-top:20px;">
                                <i-row type="flex">
                                    <i-col span="22">
                                        <div style="font-size: 33px;text-align:center;">{{applicationsData.length}}</div>
                                        <div style="margin-bottom:10px;font-size:12px;text-align:center;">申请数</div>
                                    </i-col>
                                </i-row>
                            </div>
                        </i-col>
                    </i-row>
                </div>
            </i-card>
            <i-row v-for="(item,index) in entryForManager" :key="index">
                <i-card class="layout-con" :to="item.routerTo">
                    <i-row type="flex" align="middle">
                        <i-col span="4">
                            <i-row type="flex" justify="center">
                                <Badge :count="item.badge">
                                    <i-avatar :icon="item.icon" size="large" />
                                </Badge>
                            </i-row>
                        </i-col>
                        <i-col span="20">
                            <p style="">{{item.title}}</p>
                            <p style="color: #808695; font-size:14px">{{item.description}}</p>
                        </i-col>
                    </i-row>
                </i-card>
            </i-row>
            <i-card class="layout-con" v-if="$route.query.overrideDptId" @click.native="goBack()">
                <i-row type="flex" align="middle">
                    <i-col span="4">
                        <i-row type="flex" justify="center">
                            <i-avatar icon="md-arrow-back" size="large" />
                        </i-row>
                    </i-col>
                    <i-col span="20">
                        <p style="">返回上一页面</p>
                        <p style="color: #808695; font-size:14px"></p>
                    </i-col>
                </i-row>
            </i-card>
        </i-col>
    </i-row>
</template>

<script>
import axios from 'axios';
import tableColumns from './tableColumns';
// const echarts = require("echarts");
const app = require("@/config");
export default {
    data () {
        return {
            tableColumns,
            app,
            activityData: [],
            activitySearchValue: "",
            orgInfo: {},
            time: "早上",
            dic: {
                "修改社团基本信息申请": "/manage/org/orgdetailform",
                "社团活动申请": "/manage/org/activityform"
            },
            dashBoard: {},
            pendingData: [],
            userInfo: app.userInfo,
            routers: [
                {
                    name: "OrgDetail",
                    query: {
                        tabSelect: "basicInfo"
                    }
                },
                {
                    name: "Affiliated",
                    query: {
                        tabSelect: "member"
                    }
                }
            ],
            entryForManager: {
                pending: {
                    title: "我的待办",
                    badge: 0,
                    description: "等待我处理的工作",
                    routerTo: {
                        name: "MyPending",
                        query: {}
                    },
                    icon: "md-list"
                },
                member: {
                    title: "成员管理",
                    badge: 0,
                    description: "查看所管理社团的所有成员",
                    routerTo: {
                        name: "OrgDetail",
                        query: {
                            tabSelect: "member",
                            id: this.$route.query.overrideDptId || localStorage.getItem("defaultDepartId")
                        }
                    },
                    icon: "md-person-add"
                },
                activity: {
                    title: "活动管理",
                    badge: 0,
                    description: "管理本社团的所有活动，对已经通过审核的活动可以选择开始活动。也可以在本页面下载活动签到二维码",
                    routerTo: {
                        name: "OrgDetail",
                        query: {
                            tabSelect: "activity",
                            id: this.$route.query.overrideDptId || localStorage.getItem("defaultDepartId")
                        }
                    },
                    icon: "md-flag"
                }
            },
            tableLoading: false,
            membersData: [],
            applicationsData: [],
            pager: {
                page: 1,
                pageSize: 5,
                totalRow: 0
            }
        };
    },
    mounted () {
        this.judgeTime();
        app.title = "主页";
        this.getDashBoard(this.$route.query.overrideDptId || localStorage.getItem("defaultDepartId"));
        this.getPending();
    },
    methods: {
        checkWorkflow (instanceId, stepId, actId) {
            window.open(`/manage/org/signUpSituation?instanceId=${instanceId}&stepId=${stepId}&detail=true&actId=${actId}`);
        },
        getDashBoard (id) {
            axios.post("/api/org/GetDashboard", {}, msg => {
                this.dashBoard = msg;
                axios.post("/api/security/GetOrgDetail", {id}, msg => {
                    this.orgInfo = msg.data;
                    this.getActivities();
                    axios.post("/api/security/GetUsersByDepartId", {departId: this.orgInfo.ID, pageSize: 1000}, msg => {
                        if (msg.success) {
                            this.membersData = msg.data;
                        }
                    });
                    axios.post("/api/security/GetApplicationsByDeparts", {departId: this.orgInfo.ID}, msg => {
                        if (msg.success) {
                            this.applicationsData = msg.data.filter(e => e.State === 3);
                            this.entryForManager.member.badge = this.applicationsData.length;
                        }
                    })
                })
            });
        },
        getActivities (targetPage, targetPageSize) {
            let page = targetPage || this.pager.page;
            let pageSize = targetPageSize || this.pager.pageSize;
            axios.post("/api/org/GetActByDepartId", {Id: this.orgInfo.ID, page, pageSize, name: this.activitySearchValue}, msg => {
                if (msg.success) {
                    for (let i = 0; i < msg.data.length; i++) {
                        if (msg.data[i].ActivityType !== "") {
                            msg.data[i].ActivityType = msg.data[i].ActivityType.replace(/[[\]"]/g, "").replace(/,/g, "，");
                            let reg1 = new RegExp('[r|n| ]', "g");
                            msg.data[i].ActivityType = msg.data[i].ActivityType.replace(reg1, '');
                            let reg2 = new RegExp(/\\/g, "g");
                            msg.data[i].ActivityType = msg.data[i].ActivityType.replace(reg2, '');
                        } else {
                            msg.data[i].ActivityType = "";
                        }
                    }
                    this.activityData = msg.data;
                    this.pager.totalRow = msg.totalRow;
                    console.log(msg, msg.totalRow, this.pager, this.pager.totalRow);
                    let page = 1;
                    let pageSize = this.pager.totalRow;
                    axios.post("/api/org/GetActByDepartId", {Id: this.orgInfo.ID, page, pageSize}, msg => {
                        if (msg.success) {
                            this.entryForManager.activity.badge = msg.data.filter(e => e.ApplicateState === 3).length;
                        }
                    });
                }
            });
        },
        judgeTime () {
            let day2 = new Date();
            day2.setTime(day2.getTime());
            let s2 = day2.getHours();
            if (s2 < 6) this.time = "午夜";
            else if (s2 < 12) this.time = "上午";
            else if (s2 < 14) this.time = "中午";
            else if (s2 < 18) this.time = "下午";
            else if (s2 < 24) this.time = "晚上";
        },
        getPending () {
            axios.post("/api/workflow/Pending", {}, msg => {
                this.pendingData = msg.data;
                this.entryForManager.pending.badge = msg.totalRow;
            })
        },
        dealWorkflow (instanceId, stepId, WorkflowName) {
            window.open(`${this.dic[WorkflowName]}?instanceId=${instanceId}&stepId=${stepId}&detail=false`);
        },
        navTo (url) {
            this.$router.push({name: 'OrgDetail'});
        },
        searchActivity (value) {
            this.activitySearchValue = value;
            this.getActivities();
        },
        goBack () {
            this.$router.go(-1);
        }
    }
}
</script>

<style lang="less" scoped>
    .background-purple{
        color: #ffffff;
        background: #6882da;
    }
    .layout-con {
        margin-bottom: 12px;
        color: #515a6e;
    }
    .picture {
        margin: 0px 0px 24px 0px;
        text-align: center;
        color: #808695;
        letter-spacing: 5px;
        font-size: 32px;
        border: #dcdee2 solid 3px;
        border-radius: 20px;
        height:300px;
        line-height: 300px;
    }
</style>
