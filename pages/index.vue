<script setup lang="ts">
import { DataLinkLayerWrapper, DataLinkLayer, getTextFromBytes, getBytesFromText, getHexFromBytes, getBytesFromHex } from 'audio-network-reborn';

const dataLinkLayerWrapper = new DataLinkLayerWrapper(new DataLinkLayer()); // FrameMode.Header3BytesPayloadLengthFixedAt8BytesCrc24
const bytesToTransmitInput = ref('');
const bytesReceived = ref<number[]>([]);
const receiverActive = ref(false);

const bytesToTransmit = computed(() => getBytesFromHex(bytesToTransmitInput.value));
const bytesToTransmitHex = computed(() => getHexFromBytes(bytesToTransmit.value));
const bytesReceivedHex = computed(() => getHexFromBytes(bytesReceived.value));
const isTransmitDisabled = computed(() => bytesToTransmit.value.length === 0 || bytesToTransmit.value.length > 8);

const receiverInit = () => {
    receiverActive.value = true;

    dataLinkLayerWrapper.listen({
        complete: () => {
            console.log('rx complete');
        },
        next: bytes => {
            bytesReceived.value = bytes;
        }
    });
};

const transmit = () => {
    dataLinkLayerWrapper.send(bytesToTransmit.value, {
        complete: () => {
            console.log('tx complete');
        },
        next: progress => {
            console.log('tx next', progress);
        }
    });
};

</script>

<template>
    <div>
        <h1>Send U</h1>

        <div class="section">
            <input v-model="bytesToTransmitInput" placeholder="Bytes to transmit" />

            <div class="data">{{ bytesToTransmit }}</div>
            <div class="data">{{ bytesToTransmitHex }}&nbsp;</div>

            <button @click="transmit" :disabled="isTransmitDisabled">Transmit</button>
        </div>

        <div class="section">
            <button @click="receiverInit" v-if="!receiverActive">Listen</button>
            <div v-else>Listening...</div>

            <div class="data">{{ bytesReceivedHex }}</div>
        </div>
    </div>
</template>

<style scoped lang="scss">
.section {
    padding: 16px;
    border: 1px solid lightgray;
    margin-bottom: 16px;

    data {
        font-family: monospace;
    }
}
</style>
