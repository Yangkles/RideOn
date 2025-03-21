<script setup>
import BasicHeader from '@/components/BasicHeader.vue'
import BasicFooter from '@/components/BasicFooter.vue'
import { ref, onMounted } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { useUserStore } from '@/stores/user'
import { useQnaBoardStore } from '@/stores/qnaBoard'
import { updateQnaPost } from '@/apis/qnaApi'

const router = useRouter()
const route = useRoute()
const userStore = useUserStore()
const qnaStore = useQnaBoardStore()

const title = ref('')
const content = ref('')
const tags = ref([])
const tagInput = ref('')
const titleError = ref('')
const contentError = ref('')
const isLoading = ref(false)

const postId = route.params.id

const addTag = (e) => {
  if (e.key === 'Enter' && tagInput.value.trim()) {
    if (tags.value.length < 10) {
      const trimmedTag = tagInput.value.trim()
      if (!tags.value.includes(trimmedTag)) {
        tags.value.push(trimmedTag)
        tagInput.value = ''
      } else {
        alert('이미 존재하는 태그입니다.')
      }
    } else {
      alert('태그는 최대 10개까지만 입력 가능합니다.')
    }
  }
}

const removeTag = (index) => {
  tags.value.splice(index, 1)
}

const handleKeydown = (e) => {
  if (e.key === 'Backspace' && tagInput.value === '' && tags.value.length > 0) {
    tags.value.pop()
  }
}

const resetFilters = () => {
  tags.value = []
  tagInput.value = ''
}

const validateFields = () => {
  let isValid = true

  if (!title.value.trim()) {
    titleError.value = '제목을 입력해주세요.'
    isValid = false
  } else {
    titleError.value = ''
  }

  if (!content.value.trim()) {
    contentError.value = '내용을 입력해주세요.'
    isValid = false
  } else {
    contentError.value = ''
  }

  return isValid
}

const handleSubmit = async () => {
  if (!userStore.isLoggedIn) {
    alert('로그인 후 이용해주세요.')
    router.push('/login')
    return
  }

  if (!validateFields()) {
    return
  }

  try {
    isLoading.value = true
    const postData = {
      id: postId,
      title: title.value,
      content: content.value,
      tags: tags.value,
    }

    await updateQnaPost(postData)
    alert('질문이 수정되었습니다.')
    router.push('/qnaBoard')
  } catch (error) {
    console.error('질문 수정 중 오류가 발생했습니다:', error)
    alert('질문 수정 중 오류가 발생했습니다.')
  } finally {
    isLoading.value = false
  }
}

onMounted(async () => {
  try {
    userStore.loadUser()

    const post = await qnaStore.fetchPostById(postId)

    if (userStore.user._id !== post.author._id) {
      alert('작성자만 수정할 수 있습니다.')
      router.push('/qnaBoard')
      return
    }

    title.value = post.title
    content.value = post.content
    tags.value = post.tags
  } catch (error) {
    console.error('게시글 불러오기 실패:', error)
    alert('게시글을 불러오는데 실패했습니다.')
    router.push('/qnaBoard')
  }
})
</script>

<template>
  <div class="w-full block h-full dark:bg-black9">
    <BasicHeader />
    <main class="w-[1440px] px-[93px] mx-auto pt-10 gap-4 mb-24 grid grid-cols-12">
      <section class="col-start-3 col-end-11">
        <h2 class="text-title font-bold text-black9 dark:text-black1 mb-6">질문 수정</h2>
        <form class="grid grid-cols-8 gap-4" @submit.prevent="handleSubmit">
          <article class="col-span-5 flex flex-col gap-8">
            <!-- 제목 입력 -->
            <div class="flex flex-col gap-3">
              <label for="title" class="text-body1 font-light dark:text-black1"
                >제목 <strong class="text-primaryRed">*</strong></label
              >
              <input
                v-model="title"
                type="text"
                id="title"
                class="w-full border border-black4 rounded-lg py-3 px-4 dark:bg-black8 dark:text-black1 dark:caret-black1"
                :class="{ 'border-primaryRed': titleError }"
                placeholder="제목을 입력해주세요."
              />
              <span v-if="titleError" class="text-primaryRed text-sm">{{ titleError }}</span>
            </div>

            <!-- 내용 입력 -->
            <div class="flex flex-col gap-3">
              <label for="content" class="text-body1 font-light dark:text-black1"
                >내용 <strong class="text-primaryRed">*</strong></label
              >
              <textarea
                v-model="content"
                id="content"
                class="w-full border border-black4 rounded-lg py-3 px-4 dark:bg-black8 dark:text-black1 dark:caret-black1"
                :class="{ 'border-primaryRed': contentError }"
                rows="20"
                placeholder="내용을 입력해주세요."
              ></textarea>
              <span v-if="contentError" class="text-primaryRed text-sm">{{ contentError }}</span>
            </div>
          </article>
          <article class="col-span-3 flex flex-col gap-8 self-start">
            <!-- 태그 입력 -->
            <div class="flex flex-col gap-3">
              <div class="flex flex-col gap-3">
                <label for="tags" class="text-body1 font-light dark:text-black1"
                  >태그 (최대 10개)</label
                >
                <div
                  class="flex flex-wrap gap-2 items-center w-full border border-black4 rounded-lg py-3 px-4 dark:bg-black8 dark:text-black1"
                >
                  #
                  <span
                    v-for="(tag, index) in tags"
                    :key="index"
                    class="bg-black2 text-black10 px-2 rounded dark:bg-black7 dark:text-black1"
                  >
                    {{ tag }}
                    <button type="button" @click="removeTag(index)" class="text-sm">&times;</button>
                  </span>

                  <input
                    v-model="tagInput"
                    type="text"
                    id="tags"
                    @keydown.enter.prevent="addTag"
                    @keydown="handleKeydown"
                    class="outline-none bg-transparent dark:caret-black1 dark:text-black1 flex-1"
                    placeholder="태그를 입력해주세요."
                  />
                </div>
              </div>
              <button
                v-if="tags.length > 0"
                type="button"
                class="w-full bg-black2 py-2 rounded text-black9 dark:bg-black7 dark:text-black1"
                @click="resetFilters"
              >
                <div class="flex items-center justify-center gap-2">
                  <svg
                    width="20"
                    height="20"
                    viewBox="0 0 20 20"
                    fill="none"
                    xmlns="http://www.w3.org/2000/svg"
                  >
                    <path
                      d="M17.9321 6.66797C16.6461 3.72479 13.7094 1.66797 10.2921 1.66797C5.97106 1.66797 2.4181 4.95687 2 9.16797"
                      stroke="black-10"
                      class="stroke-black10 dark:stroke-black1"
                      stroke-width="1.5"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                    />
                    <path
                      d="M14.459 6.66667H18.1256C18.4018 6.66667 18.6257 6.44281 18.6257 6.16667V2.5"
                      stroke="black-10"
                      class="stroke-black10 dark:stroke-black1"
                      stroke-width="1.5"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                    />
                    <path
                      d="M2.69336 13.332C3.97929 16.2752 6.91609 18.332 10.3333 18.332C14.6544 18.332 18.2074 15.0431 18.6255 10.832"
                      stroke="black-10"
                      class="stroke-black10 dark:stroke-black1"
                      stroke-width="1.5"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                    />
                    <path
                      d="M6.16667 13.332H2.5C2.22386 13.332 2 13.5559 2 13.832V17.4987"
                      stroke="black-10"
                      class="stroke-black10 dark:stroke-black1"
                      stroke-width="1.5"
                      stroke-linecap="round"
                      stroke-linejoin="round"
                    />
                  </svg>
                  <span class="font-bold">초기화</span>
                </div>
              </button>
            </div>

            <!-- 저장 & 취소 버튼 -->
            <div class="flex flex-col gap-4">
              <button
                type="submit"
                class="px-6 py-2 rounded text-black1 bg-primaryRed"
                :disabled="isLoading"
              >
                {{ isLoading ? '저장중...' : '저장' }}
              </button>
              <button
                type="button"
                class="px-6 py-2 rounded text-black10 bg-black2 dark:text-black1 dark:bg-black6"
                @click="$router.go(-1)"
              >
                취소
              </button>
            </div>
          </article>
        </form>
      </section>
    </main>
    <BasicFooter />
  </div>
</template>

<style scoped></style>
