<template>
  <div class="px-4 py-2" style="background-color: #eeeeee; margin-bottom: 60px;">

    <div>

      <div v-for="(item, index) in posts" :key="item.id" class="card post-card">

        <div class="card-body m-0 p-4">
          <!-- 사진url이 있거나 url시작이 http일 경우에만 실행 -->
          <img v-if="item.thumbnail && item.thumbnail.startsWith('http')" :src="item.thumbnail" />
          <div class="p-4">
            <div class="d-flex flex-row mb-5">
              <span class="fw-bold fs-1">{{ item.title }}</span>
              <button class="btn btn-sm btn-primary ms-4" @click="modifyPost(index)">수정</button>
              <button class="btn btn-sm btn-secondary ms-4" @click="removePost(index)">삭제</button>
            </div>

            <span class="text-muted fs-4">{{ item.content }}</span>
          </div>

          <div class="card-footer d-flex justify-content-between align-center py-5">
            <span class="d-flex align-items-center text-gray-600 fs-5">
              <i class="ki-duotone ki-like text-primary fs-2x me-2">
                <span class="path1"></span>
                <span class="path2"></span>
              </i>
              <span>{{ item.likes }}</span>
            </span>
            <span class="d-flex align-items-center text-gray-600 fs-5">
              <i class="ki-duotone ki-message-notif text-primary fs-2x me-2">
                <span class="path1"></span>
                <span class="path2"></span>
                <span class="path3"></span>
                <span class="path4"></span>
                <span class="path5"></span>
              </i>
              <span>{{ item.comments }}</span>
            </span>

            <span class="badge badge-light-primary px-4 py-2 fs-5">{{ item.category }}</span>
            <span class="text-muted">{{ item.createDate }}</span>
          </div>
        </div>
      </div>
    </div>
    <div class="mb-10">
      <button class="btn btn-light-primary w-100 mt-3 fs-1" @click="goToPostWrite()">
        <i class="ki-duotone ki-plus fs-2x me-2">
        </i>
        <span>새 게시물 작성</span>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

//라우터
import { useRouter } from 'vue-router';

const router = useRouter();
// 스토어 불러오기
import { storeToRefs } from 'pinia'

import { useAppStore } from '@/stores/app'
const appStore = useAppStore();
const { title } = storeToRefs(appStore);

import { usePostStore } from '@/stores/post'
const postStore = usePostStore();
const { posts, mode, selectedIndex } = storeToRefs(postStore);

// Axios 불러오기
import axios from 'axios';

onMounted(() => {
  console.log(`DocumentView::onMounted호출됨`);
  title.value = '커뮤니티'
  // 웹서버로 요청하기
  requestPostList();

})
async function requestPostList() {
  console.log(`requestPostList 호출됨`);
  try {
    const response = await axios({
      method: 'post',
      baseURL: 'http://localhost:8001',
      url: 'post/v1/list',
      data: {
      },
      timeout: 5000,
      responseType: 'json'
    })

    console.log(`응답 -> ${JSON.stringify(response.data)}`);
    posts.value = response.data.data.data;

  } catch (err) {
    console.error(`에러->${err}`);
  }
}
// 새 게시글 작성 화면
function goToPostWrite() {

  mode.value = 'add'
  router.push('/post-write');
}


function modifyPost(index) {
  console.log(`modifyPost 함수 호출됨: ${index}`)

  mode.value = 'modify'
  selectedIndex.value = index
  router.replace('/post-write', {})
}

function removePost(index) {
  console.log(`removePost함수 호출됨: ${index}`)

  const item = {
    id: posts.value[index].id
  }
  requestPostRemove(item)
}

async function requestPostRemove(item) {
  try {
    const response = await axios({
      method: 'post',
      baseURL: 'http://localhost:8001',
      url: 'post/v1/remove',
      data: item,
      timeout: 5000,
      responseType: 'json'
    })

    console.log(`응답 -> ${JSON.stringify(response.data)}`);

    requestPostList(1, 10)

  } catch (err) {
    console.error(`에러->${err}`);

  }
}
</script>
<style scoped>
.post-card {
  border: none;
  border-radius: 0.75rem;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.5);
  margin-top: 1rem;
  margin-bottom: 1rem;
  transition: transform 0.2s;

}

.post-card:hover {
  transform: translateY(-5px);
}

.post-card img {
  border-radius: 0.5rem;
  object-fit: cover;
  height: 180px;
  width: 100%;
}
</style>
