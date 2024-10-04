<template>
    <div class="absolute z-50 top-0 left-0 w-screen h-screen bg-black bg-opacity-20 grid place-items-center">
        <div class="p-4 bg-white rounded">
            <div class="pb-4">{{ props.message }}</div>
            <div class="flex justify-end gap-2">
                <button @click="ok()" class="bg-green-600 text-white p-2 rounded">OK</button>
            </div>
        </div>
    </div>
</template>

<script setup>
    const props = defineProps(["message"]);
    const emits = defineEmits(["modal-ok"]);

    function ok(){
        emits("modal-ok");
    }
</script>