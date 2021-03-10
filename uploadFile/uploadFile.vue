<template>
    <div class="upload-accessory">
        <el-upload ref="_uploadFile" :headers="headerObj" class="accessory-demo" :action="uploadUrl" name="urlFile"
            :on-remove="handleRemove" :before-upload="beforeAvatarUpload" :on-success="handleSuccess"
            :on-error="handError" :limit="limit" :file-list="fileOldLists" :on-exceed="handleExceed"
            :on-preview="handlePreview">
            <div>
                <span class="addload">添加附件</span>
                <span>
                    (<span v-show="fileTypeShow">仅支持{{fileTypeList | fileTypeListFilters}}格式，</span>
                    {{isLt}}M以内，最多{{limit}}个)</span>
            </div>
        </el-upload>
    </div>
</template>

<script>
    export default {
        props: {
            //最大上传文件个数
            limit: {
                type: Number,
                default: 6
            },
            //上传文件回显
            fileOldList: {
                type: Array,
                default: () => []
            },
            //上传文件大小,默认为10M
            isLt: {
                type: Number,
                default: 10
            },
            // 上传的文件类型,必须为数组
            fileTypeList: {
                type: Array,
                default: () => ["jpg", "png", "gif"]
            }
        },
        data() {
            return {
                headerObj: {
                    "X-Access-Token": window.localStorage.getItem("eleToken")
                },
                fileArry: [],
                fileList: "",
                uploadUrl: this.$store.state.fileUploadUrl,
                imageUrl: "",
                dialogVisible: false,
                dialogImageUrl: "",
                hideUpload: false,
            }
        },
        watch: {
            fileArry: {
                deep: true,
                immediate: true,
                handler(val) {
                    this.hideUpload = val.length == this.limit ? true : false;
                    console.log(val, val.length, this.hideUpload);
                }
            },
            fileOldLists: {
                deep: true,
                immediate: true,
                handler(val) {
                    this.fileArry = [] //每次赋值清空上次的文件id
                    this.fileList = "" //每次赋值清空上次的文件id
                    if (val.length) {
                        this.fileArry = [...this.fileArry, ...val.map(i => i.id)];
                        this.fileList = this.fileArry.join(",");
                    }
                }
            },
            fileList(val) {
                this.$emit("uploadFile", val)
            },
        },
        methods: {
            handleExceed(files, fileList) {
                this.$message.warning(
                    `当前限制选择${this.limit} 个文件，本次选择了 ${files.length
                    } 个文件，共选择了 ${files.length + fileList.length} 个文件`
                );
            },
            beforeAvatarUpload(file) {
                //获取最后一个.的位置
                console.log(file);
                let index = file.name.lastIndexOf(".");
                //获取后缀
                let fileType = file.name.substr(index + 1);
                if (this.fileTypeList.length > 0 && !this.fileTypeList.includes(fileType)) {
                    this.$message.error("抱歉,类型错误,请重新上传!");
                    return false;
                }
                const isLtM = file.size / 1024 / 1024 < this.isLt;
                const isLt0M = file.size / 1024 / 1024 == 0;
                if (isLt0M) {
                    this.$message.error("上传附件大小为0M,请重新上传!");
                    return false;
                }
                if (!isLtM) {
                    this.$message.error(`上传附件大小不能超过 ${this.isLt}MB!`);
                    return isLtM;
                }
            },
            //文件上传成功的钩子函数
            handleSuccess(res, file) {
                console.log(res, 666666666);
                this.$message({
                    type: "success",
                    message: "文件上传成功",
                    duration: 2000
                });
                this.fileArry.push(res.result.id);
                this.fileList = this.fileArry.join(",");
            },
            // 文件上传失败
            handError(err, file, fileList) {
                if (file.status != "success") {
                    this.$message({
                        type: "error",
                        message: "文件上传失败",
                        duration: 2000
                    });
                }
                console.log(err, file, fileList);
            },
            handleRemove(file, fileList) {
                if (file && file.status === "success") {
                    let id = "";
                    if (file.id) {
                        id = file.id;
                    } else {
                        id = file.response.result.id;
                    }
                    this.fileArry = this.fileArry.filter(i => {
                        return i != id;
                    });
                    this.fileList = this.fileArry.join(",");
                }

            },
            //点击文件列表中已上传文件进行下载
            handlePreview(file) {
                let id = file.id || file.response.result.id
                this.$confirm('确定下载该文件, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'info'
                }).then(() => {
                    var a = document.createElement('a');
                    var event = new MouseEvent('click');
                    a.download = file.name;
                    a.href = this.fileDownloadUrl + '?id=' + id
                    a.dispatchEvent(event);
                    console.log(a)
                }).catch(() => { })
            },
            //清空
            clearFiles(){
                this.fileArry = [] //每次赋值清空上次的文件id
                this.fileList = "" //每次赋值清空上次的文件id
                this.$refs._uploadFile.clearFiles();
            }
        },
        filters: {
            fileTypeListFilters(val) {
                return val.toString().replace('[', '').replace(']', '').replace(/,/g, '/')
            },
        },
        computed: {
            fileOldLists() {
                return this.fileOldList.filter(item => {
                    return item.id != null
                })
            },
            //仅支持格式 文字显示
            fileTypeShow() {
                return this.fileTypeList.length ? true : false
            },
            //仅支持格式 文字显示
            fileDownloadUrl() {
                return this.$store.state.fileDownloadUrl
            }
        },
    }
</script>
<style scoped>
    .upload-accessory {
        min-height: 70px;
    }

    .accessory-demo>>>.el-upload {
        height: 32px;
        color: #a2a2a2;
        border: none;
        text-align: left;
        width: 100%;
    }

    .addload {
        color: #1DA9DC;
    }
</style>