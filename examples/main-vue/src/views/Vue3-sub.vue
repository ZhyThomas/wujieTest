<template>
  <!--保活模式，name相同则复用一个子应用实例，改变url无效，必须采用通信的方式告知路由变化 -->
  <WujieVue width="100%" height="100%" name="vue3" :url="vue3Url" :plugins="plugins"></WujieVue>
</template>

<script>
import hostMap from "../hostMap";
import wujieVue from "wujie-vue2";
export default {
  data() {
    return {
      vue3Url: hostMap("//localhost:7300/") + this.$route.params.path,
      plugins: [
    {
      appendOrInsertElementHook: function (element, iframeWindow) {
        console.log(element, iframeWindow);
        if (element.offsetParent && element.offsetParent.tagName !== "BODY") {
          return;
        }
        const offsetParentDesc = Object.getOwnPropertyDescriptor(HTMLElement.prototype, "offsetParent");
        Object.defineProperties(element, {
          offsetParent: {
            configurable: true,
            get: function () {
              const offsetParent = offsetParentDesc.get.call(this);
              if ((offsetParent && offsetParent.tagName !== "BODY") || element.style.position !== "fixed") {
                return offsetParent;
              }
              return new Proxy(window.document.documentElement, {
                get: (target, propKey) => {
                  if (propKey === "parentNode") {
                    return iframeWindow.document.documentElement.parentNode;
                  }
                  const value = target[propKey];
                  const naughtySafari = typeof document.all === "function" && typeof document.all === "undefined";
                  // 只有这些场景下才需要 bind
                  if (
                    (naughtySafari
                      ? typeof value === "function" && typeof value !== "undefined"
                      : typeof value === "function") &&
                    !(value.name.indexOf("bound ") === 0 && !value.hasOwnProperty("prototype"))
                  ) {
                    const boundValue = Function.prototype.bind.call(value, target);
                    for (const key in value) {
                      boundValue[key] = value[key];
                    }
                    if (value.hasOwnProperty("prototype") && !boundValue.hasOwnProperty("prototype")) {
                      Object.defineProperty(boundValue, "prototype", {
                        value: value.prototype,
                        enumerable: false,
                        writable: true,
                      });
                    }
                    return boundValue;
                  }
                  return value;
                },
              });
            },
          },
        });
      },
    },
  ],
    };
  },
  watch: {
    $route() {
      wujieVue.bus.$emit("vue3-router-change", `/${this.$route.params.path}`);
    },
  },
  mounted(){
    console.log(123123)
  }
};
</script>

<style lang="scss" scoped></style>
