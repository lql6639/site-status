<template>
  <GlobalProvider>
    <n-scrollbar
      :content-style="{
        minHeight: '100%',
        display: 'flex',
        flexDirection: 'column',
      }"
      style="height: 100vh"
      @scroll="siteScroll"
    >
      <SiteNav />
      <Transition name="fade">
        <SiteHeader v-if="statusStore.loginStatus" />
      </Transition>
      <!-- 主内容 -->
      <main v-if="siteLoaded" id="main">
        <Transition name="fade" mode="out-in">
          <!-- 密码验证 -->
          <SiteLogin v-if="!statusStore.loginStatus" />
          <!-- 站点卡片 -->
          <SiteCards v-else />
        </Transition>
      </main>
      <SiteFooter />
      <!-- 回到顶部 -->
      <n-back-top :visibility-height="10" />
    </n-scrollbar>
  </GlobalProvider>
</template>

<script setup lang="ts">
const config = useRuntimeConfig();
const statusStore = useStatusStore();

const { setLocale } = useI18n();

// 加载状态
const siteLoaded = ref<boolean>(false);

// 验证状态
const checkSite = async () => {
  try {
    const result = await $fetch("/api/check", { method: "POST" });
    // 更改登录状态
    statusStore.loginStatus = result.code === 200;
  } catch (error) {
    console.error("error in checkSite", error);
  } finally {
    siteLoaded.value = true;
  }
};

// 页面滚动
const siteScroll = (e: Event) => {
  // 滚动高度
  const scrollTop = (e.target as HTMLElement).scrollTop;
  statusStore.scrollTop = scrollTop;
};

// 更改站点语言
const setSiteLang = (lang: string) => {
  setLocale(lang);
  useHead({ htmlAttrs: { lang } });
};

// 控制台输出
const styleTitle1 = "font-size: 20px;font-weight: 600;color: rgb(244,167,89);";
const styleTitle2 = "font-size:12px;color: rgb(244,167,89);";
const styleContent = "color: rgb(30,152,255);";
const title1 = "Lovlin_0206 の Status";
const title2 = `
 _  __   ___    _   _____   ___    ____       _
| |/ /  / _ \\  (_) |__  |  / _ \\  |  _ \\     / \\
| ' /  | | | | | |   / /  | | | | | |_) |   / _ \\
| . \\  | |_| | | |  / /_  | |_| | |  _ <   / ___ \\
|_|\\_\\  \\___/  |_| |____|  \\___/  |_| \\_\\ /_/   \\_\\`;
const content = `\n\n版本: 6.0.0\nAuthor: koizora & Github: https://github.com/lql6639`;
console.info(
  `%c${title1} %c${title2} %c${content}`,
  styleTitle1,
  styleTitle2,
  styleContent
);

// 监听站点状态
watch(
  () => statusStore.siteStatus,
  (status) => {
    const { siteTitle } = config.public;
    // 错误数据
    const isError = status === "error" || status === "warn";
    const error = statusStore.siteData?.status?.error || 0;
    const unknown = statusStore.siteData?.status?.unknown || 0;
    // 更改信息
    useHead({
      // 更改标题
      title: isError ? `( ${error + unknown} ) ` + siteTitle : siteTitle,
    });
    // 更改图标
    useFavicon(isError ? "/favicon-error.ico" : "/favicon.ico");
  },
);

// 语言更改
watch(() => statusStore.siteLang, setSiteLang);

onBeforeMount(checkSite);

onMounted(() => {
  setSiteLang(statusStore.siteLang);
});
</script>

<style lang="scss" scoped>
main {
  width: 100%;
  height: 100%;
  flex: 1;
}
</style>
