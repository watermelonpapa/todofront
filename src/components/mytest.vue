<script setup>
import {ref} from 'vue'
const inputValue = ref('')
</script>

<template>
    <div>
        <el-input v-model = "inputValue" 
        placeholder="请输入"
        style="width: 240px">
        <template #append>
            <el-button>按钮</el-button>
        </template>
        </el-input>
    </div>
</template>

<style scoped>
</style>
