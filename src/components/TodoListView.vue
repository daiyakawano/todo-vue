<script setup>
import {ref} from "vue";
import { statuses } from './const/statuses';

let items = ref(JSON.parse(localStorage.getItem("items")) || []);
let inputContent = ref();//タスクの内容
let inputLimit = ref();//タスクの期限
let inputState = ref();//タスクのステータス
let isErrMsg = ref(false);
let errMsg = ref('');
let isShowModal = ref(false);
let deleteItemId = '';
let deleteItemContent = ref();
const today = new Date();

function onEdit(id){
    let isOnEditOther = false;
    items.value.map((item) => {
        if(item.onEdit){
            isOnEditOther = true;
            return;
        }
    });
    if(isOnEditOther){
        errMsg.value = "他に編集中のタスクがあります";
        isErrMsg.value = true;
        return;
    }
    inputContent.value = items.value[id].content;
    inputLimit.value = items.value[id].limit;
    inputState.value = items.value[id].state
    items.value[id].onEdit = true;
}

function onUpdate(id){
    if(inputContent.value === "" || inputLimit.value === ""){
        errMsg.value = "タスク内容と期限を入力してください。"
        isErrMsg.value = true;
        return;
    }
    const newItem = {
        id: id,
        content: inputContent.value,
        limit: inputLimit.value,
        state: inputState.value,
        onEdit: false,
    };
    items.value.splice(id, 1, newItem);
    localStorage.setItem("items", JSON.stringify(items.value));
    isErrMsg.value = false;
}

function showDeleteModal(id){
    isShowModal.value = true;
    deleteItemId = id;
    deleteItemContent = items.value[id].content;
}

function onDeleteItem(){
    items.value.splice(deleteItemId, 1);
    items.value = items.value.map((item, index) => ({
        id: index,
        content: item.content,
        limit:item.limit,
        state: item.state,
        onEdit: item.onEdit,
    }));
    localStorage.setItem("items", JSON.stringify(items.value));
    isShowModal.value = false;
}

function onHideModal(){
    isShowModal.value = false;
}

function sortByLimit(){
    items.value.sort((a,b) => new Date(a.limit) - new Date(b.limit));
    localStorage.setItem("items", JSON.stringify(items.value));
}

function sortById(){
    items.value.sort((a,b) => a.id - b.id);
    localStorage.setItem("items", JSON.stringify(items.value));
}
</script>
<template>
    <div>
        <p v-if="isErrMsg">{{ errMsg }}</p>
        <tr>
            <th class="th-id">ID<button @click="sortById()">↓</button></th>
            <th class="th-value">やること</th>
            <th class="th-limit">期限<button @click="sortByLimit()">↓</button></th>
            <th class="th-state">状態</th>
            <th class="th-edit">編集</th>
            <th class="th-delete">削除</th>
        </tr>
        <tr
            v-for="item in items"
            :key="item.id"
            :class="{red:new Date(item.limit) < today }"
        >
            <td>{{ items.id }}</td>
            <td>
                <span v-if="!item.onEdit">{{ items.content }}</span>
                <input v-else v-model="inputContent" type="text">
            </td>
            <td>
                <span v-if="!item.onEdit">{{ items.limit }}</span>
                <input v-else v-model="inputLimit" type="date">
            </td>
            <td>
                <span v-if="!item.onEdit">{{ items.state }}</span>
                <select v-else v-model="inputState">
                    <option
                    v-for="state in statuses"
                    :key="state.id"
                    :value="state"
                    :selected="state.id = item.state.id"
                    >
                    {{ state.value }}
                </option>
                </select>
            </td>
            <td>
                <button v-if="!item.onEdit" @click="onEdit(item.id)">編集</button>
                <button v-else @click="onUpdate(item.id)">完了</button>
            </td>
            <td><button @click="showDeleteModal(item.id)">削除</button></td>
        </tr>
    </div>
    <div v-if="isShowModal" class="modal">
        <div class="modal-content">
            <p>{{ deleteItemContent }}削除してもよろしいでしょうか</p>
            <button @click="onDeleteItem()">はい</button>
            <button @click="onHideModal()">キャンセル</button>
        </div>
    </div>
</template>
<style>
.red{
    color: red;
}
</style>