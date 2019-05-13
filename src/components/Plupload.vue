<template>
  <div>
    <h4>您所选择的文件列表：</h4>
    <div id="ossfile">你的浏览器不支持flash,Silverlight或者HTML5！</div>

    <div id="uploadContainer" v-if="showUpload">
      <a id="selectfiles" href="javascript:void(0);" class="btn">选择文件</a>
      <a id="postfiles" href="javascript:void(0);" class="btn">开始上传</a>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
import { Base64 } from "@/common/js/plupload/base64";
import { util } from "@/common/js/plupload/crypto";
import { HMAC } from "@/common/js/plupload/hmac";
import { SHA1 } from "@/common/js/plupload/sha1";
import plupload from "plupload";

export default {
  data() {
    return {
      g_dirname: "landlord_video/",
      g_object_name: "",
      g_object_name_type: "",
      showUpload: true,
      showError: false
    };
  },
  mounted() {
    document.title = "上传视频";
    this.initPlUploader();
  },
  methods: {
    initPlUploader() {
      const that = this;
      console.log("initPlUploader");
      const uploader = new plupload.Uploader({
        runtimes: "html5,flash,silverlight,html4",
        browse_button: "selectfiles",
        multi_selection: false,
        container: document.getElementById("uploadContainer"),
        // flash_swf_url: '../../../../static/js/Moxie.swf',
        // silverlight_xap_url: '../../../../static/js/Moxie.xap',
        url: "http://oss.aliyuncs.com",
        filters: {
          mime_types: [{ title: "video files", extensions: "mp4,mov,mpg" }],
          max_file_size: "50mb",
          prevent_duplicates: true
        },
        init: {
          PostInit() {
            console.log("PostInit----------------------------------");
            document.getElementById("ossfile").innerHTML = "";
            document.getElementById("postfiles").onclick = function() {
              that.set_upload_param(uploader, "", false);
              return false;
            };
          },

          FilesAdded(up, files) {
            console.log("FilesAdded----------------------------------");

            plupload.each(files, file => {
              document.getElementById("ossfile").innerHTML +=
                `<div id="${file.id}">${file.name} (${plupload.formatSize(
                  file.size
                )})<b></b>` +
                '<div class="progress"><div class="progress-bar" style="width: 0%"></div></div>' +
                "</div>";
            });
          },

          BeforeUpload(up, file) {
            console.log("BeforeUpload----------------------------------");
            that.set_upload_param(up, file.name, true);
          },

          UploadProgress(up, file) {
            console.log("UploadProgress----------------------------------");

            const d = document.getElementById(file.id);
            d.getElementsByTagName("b")[0].innerHTML = `<span>${
              file.percent
            }%</span>`;
            const prog = d.getElementsByTagName("div")[0];
            const progBar = prog.getElementsByTagName("div")[0];
            progBar.style.width = `${2 * file.percent}px`;
            progBar.setAttribute("aria-valuenow", file.percent);
          },

          FileUploaded(up, file, info) {
            console.log("FileUploaded----------------------------------");

            if (info.status == 200) {
              // document.getElementById(file.id).getElementsByTagName('b')[0].innerHTML = `upload to oss success, object name:${get_uploaded_object_name(file.name)}`;
              document.getElementById(file.id).getElementsByTagName("b")[0].innerHTML = `上传成功, 文件名:${file.name}`;
              that.showUpload = false;
            } else {
              document.getElementById(file.id).getElementsByTagName("b")[0].innerHTML = info.response;
              that.showUpload = true;
            }
          },

          Error(up, err) {
            console.log("Error----------------------------------");

            // document.getElementById('console').appendChild(document.createTextNode(`\nError xml:${err.response}`));
            // document.getElementById('console').appendChild(document.createTextNode(`\n温馨提示:视频类型只能为mp4,mov,mpg, 视频大小不能超过10M`));
            that.showError = true;
          }
        }
      });
      uploader.init();
    },

    calculate_object_name(filename) {
      const suffix = this.get_suffix(filename);
      this.g_object_name = this.g_dirname + this.random_string() + suffix;
      return "";
    },
    set_upload_param(up, filename, ret) {
      const accessid = "LTAIWPvadisKkLx8e234f";
      const accesskey = "61tmGZzon3VDzSA23abdfZytIGoSPpm32rkZ";
      const host = "https://lvyaerc-gz-test.oss-cn-shenzhen.aliyuncs.com"; // 注意这里的http 或 https

      const policyText = {
        expiration: "2021-01-01T12:00:00.000Z", // 设置该Policy的失效时间，超过这个失效时间之后，就没有办法通过这个policy上传文件了
        conditions: [
          ["content-length-range", 0, 1048576000] // 设置上传文件的大小限制
        ]
      };

      const policyBase64 = Base64.encode(JSON.stringify(policyText));
      const message = policyBase64;
      const bytes = HMAC(SHA1, message, accesskey, { asBytes: true });
      const signature = util.bytesToBase64(bytes);

      this.g_object_name = this.g_dirname;
      if (filename != "") {
        this.calculate_object_name(filename);
      }
      const new_multipart_params = {
        Filename: `minprogram/${this.g_object_name}`,
        key: this.g_object_name,
        policy: policyBase64,
        OSSAccessKeyId: accessid,
        success_action_status: "200", // 让服务端返回200,不然，默认会返回204
        signature
      };
      up.setOption({
        url: host,
        multipart_params: new_multipart_params
      });

      up.start();
    },
    random_string(len) {
      len = len || 32;
      const chars =
        "ABCDEFGHIJKMNOPQRSTUVWXYZabcdefhijkmnoprstuvwxyz0123456789";
      const maxPos = chars.length;
      let pwd = "";
      for (let i = 0; i < len; i++) {
        pwd += chars.charAt(Math.floor(Math.random() * maxPos));
      }
      return pwd;
    },
    get_suffix(filename) {
      const pos = filename.lastIndexOf(".");
      let suffix = "";
      if (pos != -1) {
        suffix = filename.substring(pos);
      }
      return suffix;
    }
  }
};
</script>

<style>
.btn {
  color: #fff;
  background-color: #337ab7;
  border-color: #2e6da4;
  display: inline-block;
  padding: 6px 12px;
  margin-bottom: 0;
  font-size: 14px;
  font-weight: 400;
  line-height: 1.42857143;
  text-align: center;
  white-space: nowrap;
  text-decoration: none;
  vertical-align: middle;
  -ms-touch-action: manipulation;
  touch-action: manipulation;
  cursor: pointer;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
  background-image: none;
  border: 1px solid transparent;
  border-radius: 4px;
}
a.btn:hover {
  background-color: #3366b7;
}
.progress {
  margin-top: 2px;
  width: 200px;
  height: 14px;
  margin-bottom: 10px;
  overflow: hidden;
  background-color: #f5f5f5;
  border-radius: 4px;
  -webkit-box-shadow: inset 0 1px 2px rgba(0, 0, 0, 0.1);
  box-shadow: inset 0 1px 2px rgba(0, 0, 0, 0.1);
}
.progress-bar {
  background-color: rgb(92, 184, 92);
  background-image: linear-gradient(
    45deg,
    rgba(255, 255, 255, 0.14902) 25%,
    transparent 25%,
    transparent 50%,
    rgba(255, 255, 255, 0.14902) 50%,
    rgba(255, 255, 255, 0.14902) 75%,
    transparent 75%,
    transparent
  );
  background-size: 40px 40px;
  box-shadow: rgba(0, 0, 0, 0.14902) 0px -1px 0px 0px inset;
  box-sizing: border-box;
  color: rgb(255, 255, 255);
  display: block;
  float: left;
  font-size: 12px;
  height: 20px;
  line-height: 20px;
  text-align: center;
  transition-delay: 0s;
  transition-duration: 0.6s;
  transition-property: width;
  transition-timing-function: ease;
  width: 266.188px;
}
</style>
