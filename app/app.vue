<template>
    <div class="app-wrapper">
        <nuxt-page />
        <div
            style="position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); height: 100px; width: 1px; background-color: red;"
        ></div>
    </div>
</template>

<style>
* {
    padding: 0;
    margin: 0;
    box-sizing: border-box;
    background-color: gray;
}

.app-wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    width: 100%;
    position: relative;
}
</style>
