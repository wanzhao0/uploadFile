# uploadFile
基于vue-elem 文件上传通用组件
可以在组件内引入或全局,组件内
components: {
			uploadFile
		}
 使用:
 	      <div class="grid-content bg-purple-light">
						<div class="input-wrap">
							<el-form-item label="报告封面" prop="thumbnail" class="commonality item-title" required>
								<upload-img :limit="1" ref="_uploadImg" @uploadFile="uploadFile">
								</upload-img>
								<el-input v-model="form.thumbnail" style="display: none;"></el-input> //表单验证
							</el-form-item>
						</div>
					</div>
          //方法
            uploadFile(val) {
                this.form.attachmentId = val
            },
