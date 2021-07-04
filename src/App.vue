<template>
	<div id="app">
		<template>
			<hr>
			<el-form ref="form" label-width="80px" style="width: 100%;display: flex;">
				<div>
					<el-form-item label="文件名" required>
						<el-input v-model="file_name" placeholder="请先输入文件的下载名字,在点击上传" ></el-input>
					</el-form-item>
					<el-form-item label="备注" style="width: 500px;">
						<el-input v-model="note" placeholder="请输入文件的备注"></el-input>
					</el-form-item>
				</div>
				<div style="margin-left: 20px;">
					<el-upload class="upload-demo" drag :action="upload_host+'upload'" multiple :before-upload="beforeAvatarUpload" :data="ext_data" :on-success="success">
						<i class="el-icon-upload"></i>
						<div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
						<div class="el-upload__tip" slot="tip">文件不能超过25MB！</div>
					</el-upload>
				</div>
				<div style="width: 40%;margin-left: 20px;">
					<fieldset>
						<legend>公告：</legend>
						<h4 style="font-family: 幼圆;color: rgb(230 75 204);">由于官方API不完善等问题，暂时上传的文件只能是普通文本（可以是base64），txt，sql文件，sh脚本等，图片，exe等字节码会出错！</h4>
						<h3 style="color:rgb(206 27 134)">By: <span style="font-size: 0.1em;color: rgb(200 171 255);">技术很菜的穷B</span>  </h3>
					</fieldset>
				</div>
			</el-form>
			

			<hr>
			<el-table :data="tableData" style="width: 100%">
				<el-table-column prop="uid" label="编号" width="400"></el-table-column>
				<el-table-column prop="create_time" label="创建时间" width="200"></el-table-column>
				<el-table-column prop="file_name" label="文件名" width="350"></el-table-column>
				<el-table-column prop="note" label="备注"></el-table-column>
				<el-table-column fixed="right" label="操作" width="250">
					<template slot-scope="scope">
						<el-button @click="download(scope.row)" type="text" size="small">下载</el-button>
						<el-button @click="copyLink(scope.row)" type="text" size="small">复制下载链接</el-button>
						<el-button @click="del(scope.row)" type="text" size="small">删除</el-button>
					</template>
				</el-table-column>
			</el-table>
		</template>
	</div>
</template>

<script>
	var HOST = localStorage.getItem("host");
	if(!HOST){
		alert("请在开发人员的application的localStorage填上cloudfire的host的地址！")
	}
	export default {
		data() {
			return {
				"tableData": [],
				upload_host:HOST,
				note:'',
				file_name:'',
				ext_data:{"uid":guid(),"note":this.note,"create_time":getNowFormatDate(),file_name:this.file_name}
			}
		},
		created() {
			fetch(HOST + "./getALLStorage").then(res => {
				return res.json();
			}).then(data => {
				let d = [];
				data.forEach(e => {
					d.push(JSON.parse(e.name));
				})
				console.log(d);
				this.tableData = d;
			})
		},
		methods: {
			download: function(row) {
				console.log(row);
				//alert(HOST + "download/" + row.uid);
				window.open(HOST + "download/" + row.uid);
			},
			copyLink: function(row) {
				var input = document.createElement("input"); // 直接构建input
				input.value = HOST + "download/" + row.uid; // 设置内容
				document.body.appendChild(input); // 添加临时实例
				input.select(); // 选择实例内容
				document.execCommand("Copy"); // 执行复制
				document.body.removeChild(input); // 删除临时实例
				this.$message.success('复制成功 -->' + HOST + "download/" + row.uid);
			},
			del: function(row) {
				fetch(HOST + "./delete/" + row.uid,{
					method:"get"
				}).then(res=>{
					this.$message.success('删除成功！由于缓存影响不会立即删除,等待30秒即可。');
				});
				
			},
			beforeAvatarUpload(file) {
				const filesize = file.size / 1024 / 1024 < 25;
				if (!this.file_name) {
				  this.$message.error('请至少输入文件名！');
				  return false;
				}
				if (!filesize) {
				  this.$message.error('上传头像图片大小不能超过 2MB!');
				  return false;
				}
				this.$set(this.ext_data, "note", this.note);
				this.$set(this.ext_data, "file_name", this.file_name);
				console.log(file);
			 }
			 ,success(response, file, fileList){
				 this.$message.success('上传成功！');
				 setTimeout(()=>{
					 location.reload();
				 },1500);
			 }
		}
	}
	function guid() {
	    return (S4()+S4()+"-"+S4()+"-"+S4()+"-"+S4()+"-"+S4()+S4()+S4());
	}
	function S4() {
	    return (((1+Math.random())*0x10000)|0).toString(16).substring(1);
	}
	function getNowFormatDate() {
	        var date = new Date();
	        var seperator1 = "-";
	        var year = date.getFullYear();
	        var month = date.getMonth() + 1;
	        var strDate = date.getDate();
	        if (month >= 1 && month <= 9) {
	            month = "0" + month;
	        }
	        if (strDate >= 0 && strDate <= 9) {
	            strDate = "0" + strDate;
	        }
	        var currentdate = year + seperator1 + month + seperator1 + strDate;
	        return currentdate;
	}
</script>

<style>
	#app {
		font-family: Helvetica, sans-serif;
		text-align: center;
	}
</style>
