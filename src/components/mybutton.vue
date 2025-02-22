<script setup>
import {defineProps,defineEmits} from 'vue'
const props = defineProps(['text'])
const emit = defineEmits(['ok'])
function send(){
    emit('ok','hello')
}
</script>

<template>
    <div @click="send"  class="button">{{props.text}}</div>
</template>

<style scoped>
.button{
    width: 100px;
    height: 50px;
    background-color: blue;
}

</style>
