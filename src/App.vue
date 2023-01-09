<template>
    <input type="file" class="file-input" @change="onFileSelected($event)" />
    <br />
    <br />
    <select id="lang" name="language" :v-model="selectedLanguage">
        <option v-for="(language, index) in languages" :key="index" :value="language.value">
            {{ language.name }}
        </option>
    </select>
    <br />
    <br />
    <br />
    <h3 v-if="percentage && !textResult">
        {{ (percentage * 100).toFixed(2) + '%' }}
    </h3>
    <h3 v-if="percentage && !textResult">Running...</h3>
    <button @click="onRunTesseract()" :disabled="!file">Run Tesseract.js</button>
    <pre>{{ textResult }}</pre>
</template>

<script setup lang="ts">
import { createWorker } from 'tesseract.js';
import { ref } from 'vue';

const languages = [
    { name: 'slovak', value: 'slk' },
    { name: 'english', value: 'eng' },
];
let selectedLanguage = ref('slk');
let percentage = ref(0);
let file = ref();
let wholeResult = ref();
let textResult = ref();

function onFileSelected(event) {
    file.value = event.target.files[0];
}

async function onRunTesseract() {
    textResult.value = '';
    await doOCR();
}

async function doOCR() {
    console.log(file, selectedLanguage);

    const worker = await createWorker({
        logger: (m) => {
            percentage.value = m.progress;
            console.log('progress ->', m);
        },
    });
    await worker.load();
    await worker.loadLanguage(selectedLanguage.value);
    await worker.initialize(selectedLanguage.value);
    wholeResult.value = await worker.recognize(file.value);

    textResult.value = JSON.parse(JSON.stringify(wholeResult.value.data.text));
    console.log('result from tesseract ->', wholeResult.value);

    await worker.terminate();
}
</script>
