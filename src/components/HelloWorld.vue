<template>
  <div class="header">
    <div class="buttomWarper">
      <div id="clear" class="buttom" @click="cleardata">清除内容</div>
      <div id="copy" class="buttom" @click="copydata">复制结果</div>
    </div>
  </div>
  <div class="warper">
    <textarea id="inputArea" v-model="inputdata" />
    <textarea id="outputArea" v-model="outputdata" ref="outputArea" />
  </div>
</template>

<script>
import { defineComponent, ref, watch } from "vue";
import { ElNotification } from "element-plus";

export default defineComponent({
  components: {
    ElNotification,
  },
  setup() {
    const inputdata = ref("");
    const outputdata = ref("");
    const outputArea = ref("");
    watch(inputdata, () => {
      readAndProcess();
    });
    const readAndProcess = () => {
      const rawdata = inputdata.value.trim();
      if (rawdata) {
        const objdata = JSON.parse(rawdata);
        const simplifieddata = simplifyData(objdata);
        outputdata.value = deepstruct(simplifieddata, 0);
      } else {
        outputdata.value = "";
      }
    };
    const simplifyData = (data) => {
      if (data instanceof Array) {
        let ret = [];
        if (data[0] instanceof Object || data.length >= 3) {
          ret[0] = simplifyData(data[0]);
        } else {
          ret = data;
        }
        return ret;
      } else if (data instanceof Object) {
        let ret = {};
        const keys = Object.keys(data);
        keys.forEach((key) => {
          ret[key] = simplifyData(data[key]);
        });
        return ret;
      } else {
        return data;
      }
    };
    const depthspace = (depth) => {
      let ret = "";
      for (let i = 0; i < depth; i++) {
        ret += "    ";
      }
      return ret;
    };
    const deepstruct = (data, depth) => {
      let ret = "";
      if (data instanceof Array) {
        ret += "[\n";
        for (let i = 0; i < data.length; i++) {
          ret += depthspace(depth + 1);
          ret += deepstruct(data[i], depth + 1);
          if (i != data.length - 1) {
            ret += ",";
          }
          ret += "\n";
        }
        ret += depthspace(depth);
        ret += "]";
      } else if (data instanceof Object) {
        ret += "{\n";
        const keys = Object.keys(data);
        for (let i = 0; i < keys.length; i++) {
          ret += depthspace(depth + 1);
          ret += `\"${keys[i]}\":`;
          ret += deepstruct(data[keys[i]], depth + 1);
          if (i != keys.length - 1) {
            ret += ",";
          }
          ret += "\n";
        }
        ret += depthspace(depth);
        ret += "}";
      } else if (typeof data == "string") {
        console.log("String,", data);
        ret += `\"${data}\"`;
      } else {
        ret += data;
      }
      return ret;
    };

    const cleardata = (e) => {
      e.target.style.backgroundColor = "#ffe600";
      setTimeout(() => {
        e.target.style.backgroundColor = "#409eff";
      }, 80);
      inputdata.value = "";
    };
    const copydata = (e) => {
      e.target.style.backgroundColor = "#ffe600";
      setTimeout(() => {
        e.target.style.backgroundColor = "#409eff";
      }, 80);
      outputArea.value.select();
      try {
        document.execCommand("copy");
        ElNotification({
          title: "成功",
          message: "复制成功！",
          type: "success",
          duration: 800,
          position: "top-right",
        });
      } catch (e) {
        ElNotification({
          title: "失败",
          message: "复制失败！",
          type: "error",
          duration: 800,
          position: "top-right",
        });
      }
    };
    return {
      inputdata,
      outputdata,
      outputArea,
      readAndProcess,
      simplifyData,
      cleardata,
      copydata,
      depthspace,
      deepstruct,
    };
  },
});
</script>

<style scoped lang="scss">
.header {
  height: 60px;
  padding: 0px 40px 10px;
  min-width: 400px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: dimgray;
  .buttomWarper {
    display: flex;
    justify-content: center;
    width: 100%;
    .buttom {
      background-color: #409eff;
      border: #25619d 1px solid;
      border-radius: 7px;
      color: white;
      width: 50%;
      min-width: 150px;
      height: 45px;
      margin-top: 15px;
      display: flex;
      justify-content: center;
      align-items: center;
      font-size: 24px;
      user-select: none;
      &:hover {
        background-color: #60aeff;
      }
    }
    #clear {
      margin-right: 10px;
    }
    #copy {
      margin-left: 10px;
    }
  }
}
.warper {
  display: flex;
  padding: 0 40px 20px 40px;
  background-color: dimgray;
  height: calc(100% - 90px);
  min-width: 400px;
  #inputArea {
    width: 50%;
    height: calc(100% - 26px);
    background-color: bisque;
    display: inline-block;
    margin-right: 10px;
    border-radius: 5px;
    resize: none;
  }
  #outputArea {
    width: 50%;
    height: calc(100% - 26px);
    background-color: floralwhite;
    display: inline-block;
    margin-left: 10px;
    border-radius: 5px;
    resize: none;
  }
}
</style>