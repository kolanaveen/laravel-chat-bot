<script setup>
import {nextTick, onMounted, ref, watch, watchEffect} from "vue";
import {usePage} from "@inertiajs/vue3";

const modal = ref(false)

const switchVisibility = () => {
  modal.value = modal.value === false;
}

const currentUser = ref(usePage().props.auth.user)
const messages = ref([]);
const newMessage = ref("");
const messagesContainer = ref(null);
const isFriendTyping = ref(false);
const isFriendTypingTimer = ref(null);
const friendId = currentUser.value.id === 1 ? 2 : 1;

const sendTypingEvent = () => {
  Echo.private(`chat.${friendId}`).whisper("typing", {
    userID: currentUser.value.id,
  });
};

const sendMessage = () => {
  if (newMessage.value.trim() !== "") {
    axios
        .post(`/messages/${friendId}`, {
          message: newMessage.value,
        })
        .then((response) => {
          messages.value.push(response.data);
          newMessage.value = "";
        });
  }
};
watch(
    messages,
    () => {
      nextTick(() => {
        messagesContainer.value.scrollTo({
          top: messagesContainer.value.scrollHeight,
          behavior: "smooth",
        });
      });
    },
    { deep: true }
);

onMounted(() => {
  axios.get(`/messages/${friendId}`)
      .then(res => {
        messages.value = res.data
      });

  Echo.private(`chat.${currentUser.value.id}`)
      .listen("MessageSent", (response) => {
        messages.value.push(response.message);
      })
      .listenForWhisper("typing", (response) => {
        isFriendTyping.value = response.userID === friendId;

        if (isFriendTypingTimer.value) {
          clearTimeout(isFriendTypingTimer.value);
        }

        isFriendTypingTimer.value = setTimeout(() => {
          isFriendTyping.value = false;
        }, 1000);
      });
})

</script>

<template>
  <button @click="switchVisibility"
          class="fixed bottom-4 right-4 inline-flex items-center justify-center text-sm font-medium disabled:pointer-events-none disabled:opacity-50 border rounded-full w-16 h-16 bg-[#26a69a] hover:bg-[#26a69a] m-0 cursor-pointer border-gray-200 bg-none p-0 normal-case leading-5 hover:text-gray-900"
          type="button" aria-haspopup="dialog" aria-expanded="false" data-state="closed">
    <svg xmlns=" http://www.w3.org/2000/svg" width="30" height="40" viewBox="0 0 24 24" fill="none"
         stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"
         class="text-white block border-gray-200 align-middle">
      <path d="m3 21 1.9-5.7a8.5 8.5 0 1 1 3.8 3.8z" class="border-gray-200">
      </path>
    </svg>
  </button>


  <div style="box-shadow: 0 0 #0000, 0 0 #0000, 0 1px 2px 0 rgb(0 0 0 / 0.05);" v-if="modal"
       class="fixed bottom-[calc(4rem+1.5rem)] right-0 mr-4 bg-white p-6 rounded-lg border border-[#e5e7eb] w-[440px] h-[634px]">

    <!-- Heading -->
    <div class="flex flex-col space-y-1.5 pb-6">
      <h2 class="font-semibold text-lg tracking-tight">Chatbot</h2>
      <p class="text-sm text-[#6b7280] leading-3">Powered by Mendable and Vercel</p>
    </div>


<!--    &lt;!&ndash; Chat Container &ndash;&gt;-->
<!--    <div ref="messagesContainer" class="pr-4 h-[474px]" style="min-width: 100%; display: table;">-->
<!--      &lt;!&ndash; Chat Message AI &ndash;&gt;-->
<!--      <div class="flex gap-3 my-4 text-gray-600 text-sm flex-1"><span-->
<!--          class="relative flex shrink-0 overflow-hidden rounded-full w-8 h-8">-->
<!--          <div class="rounded-full bg-gray-100 border p-1">-->
<!--            <svg stroke="none" fill="black" stroke-width="1.5" viewBox="0 0 24 24" aria-hidden="true" height="20"-->
<!--                 width="20" xmlns="http://www.w3.org/2000/svg">-->
<!--              <path stroke-linecap="round" stroke-linejoin="round"-->
<!--                    d="M9.813 15.904L9 18.75l-.813-2.846a4.5 4.5 0 00-3.09-3.09L2.25 12l2.846-.813a4.5 4.5 0 003.09-3.09L9 5.25l.813 2.846a4.5 4.5 0 003.09 3.09L15.75 12l-2.846.813a4.5 4.5 0 00-3.09 3.09zM18.259 8.715L18 9.75l-.259-1.035a3.375 3.375 0 00-2.455-2.456L14.25 6l1.036-.259a3.375 3.375 0 002.455-2.456L18 2.25l.259 1.035a3.375 3.375 0 002.456 2.456L21.75 6l-1.035.259a3.375 3.375 0 00-2.456 2.456zM16.894 20.567L16.5 21.75l-.394-1.183a2.25 2.25 0 00-1.423-1.423L13.5 18.75l1.183-.394a2.25 2.25 0 001.423-1.423l.394-1.183.394 1.183a2.25 2.25 0 001.423 1.423l1.183.394-1.183.394a2.25 2.25 0 00-1.423 1.423z">-->
<!--              </path>-->
<!--            </svg>-->
<!--          </div>-->
<!--        </span>-->
<!--        <p class="leading-relaxed"><span class="block font-bold text-gray-700">AI </span>Hi,-->
<!--          how can I help you today?-->
<!--        </p>-->
<!--      </div>-->

<!--      &lt;!&ndash;  User Chat Message &ndash;&gt;-->
<!--      <div class="flex gap-3 my-4 text-gray-600 text-sm flex-1"><span-->
<!--          class="relative flex shrink-0 overflow-hidden rounded-full w-8 h-8">-->
<!--          <div class="rounded-full bg-gray-100 border p-1"><svg stroke="none" fill="black" stroke-width="0"-->
<!--                                                                viewBox="0 0 16 16" height="20" width="20"-->
<!--                                                                xmlns="http://www.w3.org/2000/svg">-->
<!--              <path-->
<!--                  d="M8 8a3 3 0 1 0 0-6 3 3 0 0 0 0 6Zm2-3a2 2 0 1 1-4 0 2 2 0 0 1 4 0Zm4 8c0 1-1 1-1 1H3s-1 0-1-1 1-4 6-4 6 3 6 4Zm-1-.004c-.001-.246-.154-.986-.832-1.664C11.516 10.68 10.289 10 8 10c-2.29 0-3.516.68-4.168 1.332-.678.678-.83 1.418-.832 1.664h10Z">-->
<!--              </path>-->
<!--            </svg></div>-->
<!--        </span>-->
<!--        <p class="leading-relaxed"><span class="block font-bold text-gray-700">You </span>fewafef</p>-->
<!--      </div>-->
<!--    </div>-->
<!--    &lt;!&ndash; Input box  &ndash;&gt;-->
<!--    <div class="flex items-center pt-0">-->
<!--      <form class="flex items-center justify-center w-full space-x-2">-->
<!--        <input-->
<!--            v-model="newMessage"-->
<!--            @keydown="sendTypingEvent"-->
<!--            @keyup.enter="sendMessage"-->
<!--            class="flex h-10 w-full rounded-md border border-[#e5e7eb] px-3 py-2 text-sm placeholder-[#6b7280] focus:outline-none focus:ring-2 focus:ring-[#9ca3af] disabled:cursor-not-allowed disabled:opacity-50 text-[#030712] focus-visible:ring-offset-2"-->
<!--            placeholder="Type your message" value="">-->
<!--        <button-->
<!--            class="inline-flex items-center justify-center rounded-md text-sm font-medium text-[#f9fafb] disabled:pointer-events-none disabled:opacity-50 bg-[#26a69a] hover:bg-[#111827E6] h-10 px-4 py-2">-->
<!--          Send-->
<!--        </button>-->
<!--      </form>-->
<!--    </div>-->
    <div>
      <div class="flex flex-col justify-end h-80">
        <div ref="messagesContainer" class="p-4 overflow-y-auto max-h-fit">
          <div
              v-for="message in messages"
              :key="message.id"
              class="flex items-center mb-2"
          >
            <div
                v-if="message.sender_id === currentUser.id"
                class="p-2 ml-auto text-white bg-blue-500 rounded-lg"
            >
              {{ message.text }}
            </div>
            <div v-else class="p-2 mr-auto bg-gray-200 rounded-lg">
              {{ message.text }}
            </div>
          </div>
        </div>
      </div>
      <div class="flex items-center">
        <input
            type="text"
            v-model="newMessage"
            @keydown="sendTypingEvent"
            @keyup.enter="sendMessage"
            placeholder="Type a message..."
            class="flex-1 px-2 py-1 border rounded-lg"
        />
        <button
            @click="sendMessage"
            class="px-4 py-1 ml-2 text-white bg-blue-500 rounded-lg"
        >
          Send
        </button>
      </div>
      <small v-if="isFriendTyping" class="text-gray-700">
        is typing...
      </small>
    </div>
  </div>
</template>