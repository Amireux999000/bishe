<template>
    <div class="project-space-files">
        <wrapper-content :showHeader="false">
            <div class="content-wrapper">
                <div class="content-item log-list">
                    <div class="header">
                        <span class="title">我的文件</span>
                        <div class="header-actions">
                            <a-button id="upload-file" icon="up-circle" type="dashed">上传</a-button>
                        </div>
                    </div>
                    <div class="list-content">
                        <a-list :loading="loading">
                            <a-list-item class="list-item-title">
                                <a-list-item-meta>
                                    <div class="muted" slot="title">名称</div>
                                </a-list-item-meta>
                                <div class="other-info muted">
                                    <div class="info-item"  style="padding-right: 20px;"><span>大小</span></div>
                                    <div class="info-item"><span>创建日期</span></div>
                                    <div class="info-item">
                                        <span>创建人</span>
                                    </div>
                                </div>
                                <span slot="actions" :key="item" v-for="item in 3">
                                    <span>位</span>
                                </span>
                            </a-list-item>
                            <a-list-item class="list-item" :key="index" v-for="(item, index) in files">
                                <a-list-item-meta>
                                    <a-avatar slot="avatar" shape="square" :src="item.fileUrl" v-if="item.extension === 'html'" style="">
                                        <a-icon type="code" />
                                    </a-avatar>
                                    <a-avatar slot="avatar" shape="square" :src="item.fileUrl" v-else-if="item.extension === 'pdf'">
                                        <a-icon type="file-pdf"/>
                                    </a-avatar>
                                    <a-avatar slot="avatar" shape="square" :src="item.fileUrl" v-else-if="item.extension === 'txt'">
                                        <a-icon type="file-text"/>
                                    </a-avatar>
                                    <a-avatar slot="avatar" shape="square" :src="item.fileUrl" v-else-if="item.extension === 'doc' || item.extension === 'docx'">
                                        <a-icon type="file-word"/>
                                    </a-avatar>
                                    <a-avatar slot="avatar" shape="square" :src="item.fileUrl" v-else-if="item.extension === 'jpg'">
                                        <a-icon type="file-jpg"/>
                                    </a-avatar>
                                    <a-avatar slot="avatar" shape="square" :src="item.fileUrl" v-else-if="item.extension === 'xls' || item.extension === 'xlsx'">
                                        <a-icon type="file-excel"/>
                                    </a-avatar>
                                    <a-avatar slot="avatar" shape="square" :src="item.fileUrl" v-else-if="item.extension === 'mp3'">
                                        <a-icon type="customer-service"/>
                                    </a-avatar>
                                    <a-avatar slot="avatar" shape="square" :src="item.fileUrl" v-else-if="item.extension === 'zip'">
                                        <a-icon type="folder-open"/>
                                    </a-avatar>
                                    <a-avatar slot="avatar" shape="square" :src="item.fileUrl" v-else>
                                        <a-icon type="file" />
                                    </a-avatar>
                                    <div slot="title">
                                        <a-tooltip :mouseEnterDelay="0.3">
                                            <template slot="title">
                                                <span>{{item.originalName}}</span>
                                            </template>
                                            <a-input
                                                    :ref="`inputTitle${index}`"
                                                    :auto-focus="true"
                                                    v-model="item.name"
                                                    v-show="item.editing"
                                                    @pressEnter="onCellChange(item)"
                                                    @blur="onCellChange(item)"></a-input>
                                            <!--   <a class="text-default" target="_blank" :href="item | showPreviewUrl"
                                                  v-show="!item.editing">{{item.fullName}}</a>-->
                                            <a class="text-default" v-show="!item.editing" @click="seeBox(item)">{{item.originalName}}</a>
                                        </a-tooltip>
                                    </div>
                                </a-list-item-meta>
                                <div class="other-info muted">
                                    <div class="info-item" style="padding-right: 20px;">
                                        <span>{{item.length}}</span>
                                    </div>
                                    <div class="info-item">
                                        <a-tooltip :title="item.createTime">
                                            <span>{{ formatTime(item.createTime) }}</span>
                                        </a-tooltip>
                                    </div>
                                    <div class="info-item">
                                        <span>{{item.createUser}}</span>
                                    </div>
                                </div>
                                <span slot="actions">
                                    <a-tooltip title="下载">
                                        <a class="muted" target="_blank" :href="item.fileUrl"><a-icon type="download"/></a>
                                    </a-tooltip>
                                </span>
                                <span slot="actions" @click="editFile(item,index)">
                                    <a-tooltip title="编辑">
                                        <a-icon type="edit"/>
                                    </a-tooltip>
                                </span>
                                <a class="muted" slot="actions">
                                    <a-dropdown :trigger="['click']" placement="bottomCenter">
                                        <a-tooltip :mouseEnterDelay="0.5">
                                            <template slot="title">
                                                <span>更多操作</span>
                                            </template>
                                            <a class="action-item muted">
                                                <a-icon type="down"/>
                                            </a>
                                        </a-tooltip>
                                        <a-menu class="field-right-menu"
                                                @click="copyUrl(item.fileUrl)"
                                                slot="overlay">
                                            <a-menu-item key="copy" v-clipboard="item.fileUrl" data-clipboard-text="我是可以复制的内容，啦啦啦啦">
                                                <a-icon type="link"/>
                                                <span>复制链接</span>
                                            </a-menu-item>
                                        </a-menu>
                                    </a-dropdown>
                                </a>
                            </a-list-item>
                            <div v-if="showLoadingMore" slot="loadMore"
                                 :style="{ textAlign: 'center', marginTop: '12px', height: '32px', lineHeight: '32px' }">
                                <a-spin v-if="loadingMore"/>
                                <a-button v-else @click="onLoadMore">查看更多文件</a-button>
                            </div>
                        </a-list>
                    </div>
                </div>
            </div>
        </wrapper-content>
        <box v-model="showInviteMember" v-if="showInviteMember" :seeUrl="seeUrl"></box>
    </div>
</template>

<script>
    import {mapState} from 'vuex'
    import {checkResponse} from "../../assets/js/utils";
    import {relativelyTime} from "../../assets/js/dateTime";
    import pagination from "@/mixins/pagination";
    import {notice} from "../../assets/js/notice";
    import {getFiles, fileRename} from "../../api/mock";
    import box from '../../components/file/box'

    export default {
        name: "project-space-files",
        components: {
            'box':box
        },
        mixins: [pagination],
        data() {
            return {
                code: this.$route.params.code,
                loading: true,
                showLoadingMore: false,
                loadingMore: false,
                project: {},
                currentFileIndex: {},
                files: [],
                showInviteMember: false,
                seeUrl: 'http://ppkn5nh6t.bkt.clouddn.com/upload/20190410/dbf66652effa4b309d98b00946043690.jpeg',
            }
        },
        computed: {
            ...mapState({
                uploader: state => state.common.uploader,

            }),
        },
        watch: {
            uploader: {
                handler(newVal, oldVal) {
                    //监听是否有上传文件行为
                    const files = newVal.fileList;
                    /* const index = files.findIndex(item => item.projectName == this.project.name);
                     if (index !== -1) {
                         this.getFiles();
                     }*/
                    this.getFiles();
                },
                deep: true
            }
        },
        created() {
            this.getFiles();
        },
        mounted() {
            setTimeout(() => {
                this.uploader.assignBrowse(document.getElementById('upload-file'));
            }, 500)
        },
        methods: {
            getFiles(reset = true) {
                let app = this;
                if (reset) {
                    this.pagination.page = 1;
                    this.pagination.pageSize = 50;
                    this.showLoadingMore = false;
                }
                app.requestData.projectCode = this.code;
                app.requestData.deleted = 0;
                this.loading = true;
                getFiles(app.requestData).then(res => {
                    this.loading = false;
                    if (reset) {
                        this.files = [];
                    }
                    res.data.list.forEach((v) => {
                        v.editing = false;
                    });
                    app.files = app.files.concat(res.data.list);
                    app.pagination.total = res.data.total;
                    app.showLoadingMore = app.pagination.total > app.files.length;
                    app.loading = false;
                    app.loadingMore = false
                })
            },
            onLoadMore() {
                this.loadingMore = true;
                this.pagination.page++;
                this.getFiles(false);
            },
            editFile(file, index) {
                let app = this;
                this.files.forEach((v) => {
                    v.editing = false;
                });
                this.files[index].editing = true;
                this.$nextTick(() => {
                    app.$refs[`inputTitle${index}`][0].focus();
                });
                this.currentFileIndex = index;
            },
            seeBox(file) {
                var file_url = file.fileUrl;
                this.showInviteMember = true;
                this.seeUrl = file_url;
            },
            copyUrl(url) {
                notice({
                    title: '复制成功',
                }, 'notice', 'success');
            },
            onCellChange(file) {
                let currentFile = this.files[this.currentFileIndex];
                this.files.forEach((v) => {
                    v.editing = false;
                });
                const fullName = `${file.name}.${file.extension}`;
                if (fullName != currentFile.originalName) {
                    fileRename({fullName: fullName, fileId: currentFile.id}).then(res => {
                        const result = checkResponse(res);
                        if (!result) {
                            return false;
                        }
                        currentFile.name = file.name;
                        currentFile.originalName = fullName;
                        notice({
                            title: '重命名成功',
                        }, 'notice', 'success');
                    });
                }
            },
            formatTime(time) {
                return relativelyTime(time);
            },
            formatSize(size) {
                let type = 'KB';
                size = size / 1024;
                if (size >= 1024) {
                    size /= 1024;
                    type = 'MB';
                }
                return `${size.toFixed(2)} ${type}`;
            },
        }
    }
</script>

<style lang="less">

    .project-space-files {
        .project-navigation {
            top: 65px;
            z-index: 4;
        }

        .wrapper-main {
            background: initial;

        }

        .layout-content {
            padding: 12px;
            width: 1100px;
            margin: 12px auto auto;
            background: initial;
            display: flex;
            flex-direction: row;
            justify-content: space-between;

            .content-item {
                background: #fff;
                padding: 6px 0 18px 0;
                border-radius: 4px;

                .header {
                    padding: 12px 20px 6px 20px;
                    display: flex;
                    justify-content: space-between;

                    .title {
                        font-size: 18px;
                    }
                }
            }

            .content-wrapper {
                width: 100%;

                .log-list {
                    background: #fff;

                    .list-content {

                        .list-item-title {
                            padding: 10px 20px;

                            .ant-list-item-action {
                                li {
                                    color: #fff;
                                }

                                em {
                                    width: 0;
                                }
                            }
                        }

                        .list-item {
                            border-bottom: none;
                            margin-bottom: 2px;
                            /*border-bottom: 1px solid #f5f5f5;*/
                            padding: 10px 20px;
                            transition: background-color 218ms;

                            &:hover {
                                background-color: #f5f5f5;
                            }

                            .ant-list-item-meta-title {
                                overflow: hidden;
                                text-overflow: ellipsis;
                                white-space: nowrap;
                                position: relative;
                                margin-bottom: 0;
                                line-height: 42px;
                            }

                            .ant-list-item-action {
                                em {
                                    width: 0;
                                }
                            }
                        }

                        .other-info {
                            display: flex;

                            .info-item {
                                display: flex;
                                flex-direction: column;
                                padding-left: 0;
                                width: 100px;
                                text-align: right;
                            }

                            .schedule {
                                width: 250px;
                            }
                        }
                    }
                }
            }

            .content-right {
                width: 325px;
            }
        }
    }
</style>
