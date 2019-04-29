<template>
    <div>
        <div class="md-modal modal-msg md-modal-transition" :class="{'md-show': modelShow}">
            <div class="md-modal-inner">
            <div class="md-top">
                <button class="md-close" @click="closeModel">Close</button>
            </div>
            <div class="md-content">
                <div class="confirm-tips">
                    <slot name="message"></slot>
                </div>
                <div class="btn-wrap">
                    <slot name="btnGroup"></slot>
                </div>
            </div>
            </div>
        </div>
        <div class="md-overlay" v-if="modelShow" @click="closeModel"></div>
    </div>
</template>
<script>
export default {
    name: 'CommonModel',
    props: ["modelShow"],
    methods: {
        closeModel () {
            this.$emit('closeModel');
        }
    }
}
</script>
<style lang='stylus' scoped>
</style>