<template>
    <div class="OverViewHead-content">
        <div class=OverViewHead-logo>
            <img src="#" alt="logo">
            <span>智慧水务</span>
        </div>
        <div class="OverViewHead-title">
            <h2>智慧水务综合管理平台</h2>
        </div>
        <div class="OverViewHead-menu">
            <span>聚类图</span>
            <span>向下箭头</span>
            <span>管理员icon</span>
            <span>超级管理员</span>
            <span>向下箭头</span>
        </div>
    </div>
</template>

<script>
export default {
  name: 'OverViewHead',
  components: {
  }
}
</script>

<style spend>
html, body {
    margin: 0;
    padding: 0;
}
.OverViewHead-content {
    display: flex;
    height: 10%;
}

.OverViewHead-logo {
    width: 33.33%;  
}

.OverViewHead-title{
    width: 33.33%;
    line-height: calc(100% - 71.5px);
}

.OverViewHead-menu {
    width: 33.33%;
}
</style>