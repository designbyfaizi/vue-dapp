<template>
    <input
        class="bg-gray-800 p-1 rounded-md outline-none border-2 border-gray-700 focus:border-blue-800 transition-all duration-150 focus:placeholder-blue-400"
        :type="type"
        :placeholder="placeholder"
        :value="value"
        v-on:change="onChange"
    />
</template>

<script>
export default {
    props:['placeholder', 'type', "value", "onChange"]
};
</script>

<style lang="scss" scoped></style>
