<script setup lang="ts">
import { ref, onBeforeMount } from "vue";
import { fetchy } from "../../utils/fetchy";

const emit = defineEmits(["refreshRequests", "refreshPending", "refreshFriends"]);
const loaded = ref(false);
const requests = ref([]);
const pending = ref([]);

async function getRequests() {
    let response;
    try {
        response = await fetchy(`/api/friend/requests`, "GET");
    } catch (e) {
        console.log(e);
        return;
    }
    requests.value = response;
}

async function acceptRequest(from: string) {
    try {
        await fetchy(`/api/friend/accept/${from}`, "PUT");
    } catch (e) {
        console.log(e);
        return;
    }
    emit("refreshRequests");
    emit("refreshFriends");
    await getRequests();
}

async function rejectRequest(from: string) {
    try {
        await fetchy(`/api/friend/reject/${from}`, "PUT");
    } catch (e) {
        console.log(e);
        return;
    }
    emit("refreshRequests");
    await getRequests();
}

async function getPending() {
    let response;
    try {
        response = await fetchy(`/api/friend/pending`, "GET");
    } catch (e) {
        console.log(e);
        return;
    }
    pending.value = response;
}

async function removeRequest(to: string) {
    try {
        await fetchy(`/api/friend/requests/${to}`, "DELETE")
    } catch (e) {
        console.log(e);
        return;
    }
    emit("refreshPending");
    await getPending();
}

onBeforeMount(async () => {
  await getRequests();
  await getPending();
  loaded.value = true;
});

</script>

<template>
    <h2> Requests Awaiting Approval </h2>
    <section v-if="loaded">
        <p v-for="request in requests" :key="request._id">
            {{ request.from }};
            <button @click="acceptRequest(request.from)">Accept</button>
            <button @click="rejectRequest(request.from)">Reject</button>
        </p>
    </section>
    <h2> Pending Requests </h2>
    <section v-if="loaded">
        <p v-for="pend in pending" :key="pend._id">
            {{ pending.to }}
            <button @click="removeRequest(request.to)">Remove</button>
        </p>
    </section>
</template>

<style scoped>

</style>