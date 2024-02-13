<template>
  <RouterLink
    to="/about"
    class="bg-green-400 px-3 py-1 rounded-md hover:bg-green-500"
    >跳轉</RouterLink
  >
  <form
    action="#"
    class="border max-w-96 p-4 border-gray-400 mt-4"
    @submit.prevent="onSubmit"
  >
    <div class="flex items-center">
      <label for="apiId">API ID</label>
      <input
        type="text"
        id="apiId"
        class="border p-1 flex-1 ml-4 border-gray-400"
        v-model="apiId"
      />
    </div>
    <div class="flex items-center mt-4">
      <label for="apiHash">API HASH</label>
      <input
        type="text"
        id="apiHash"
        class="border p-1 flex-1 ml-4 border-gray-400"
        v-model="apiHash"
      />
    </div>
    <div class="flex items-center mt-4">
      <label for="phoneNumber">PHONE NUMBER</label>
      <input
        type="text"
        id="phoneNumber"
        class="border p-1 flex-1 ml-4 border-gray-400"
        v-model="phoneNumber"
      />
    </div>
    <div class="flex justify-end mt-4">
      <button
        type="button"
        class="bg-green-400 px-3 py-1 rounded-md hover:bg-green-500"
        @click.prevent="sendCode"
      >
        發送確認碼
      </button>
    </div>
    <!-- <div class="flex items-center mt-4">
      <label for="password">PASSWORD</label>
      <input
        type="text"
        id="password"
        class="border p-1 flex-1 ml-4 border-gray-400"
        v-model="password"
      />
    </div> -->
    <div class="flex items-center mt-4">
      <label for="phoneCode">PHONE CODE</label>
      <input
        type="text"
        id="phoneCode"
        class="border p-1 flex-1 ml-4 border-gray-400"
        v-model="phoneCode"
      />
    </div>
    <div class="flex justify-end mt-4">
      <button
        type="submit"
        class="bg-green-400 px-3 py-1 rounded-md hover:bg-green-500"
      >
        連接
      </button>
    </div>
  </form>
</template>

<script setup lang="ts">
import { TelegramClient, Api } from "telegram";
import { StringSession } from "telegram/sessions";
import { onBeforeUnmount, onMounted } from "vue";
import { ref } from "vue";
import { RouterLink } from "vue-router/auto";

const apiId = ref("");
const apiHash = ref("");
const phoneNumber = ref("");
const password = ref("");
const phoneCode = ref("");
const phoneCodeHash = ref("");

// const userAuthParamCallback (param) {
//   return async function (){
//     return await new (resolve => {
//       resolve(param)
//     })()
//   }
// }

let client: TelegramClient;

const func = (event: any) => {
  console.log(event);
};

const sendCode = async () => {
  const session = new StringSession("");

  client = new TelegramClient(session, +apiId.value, apiHash.value, {});

  await client.connect();

  const res = await client.sendCode(
    { apiId: +apiId.value, apiHash: apiHash.value },
    phoneNumber.value
  );

  phoneCodeHash.value = res.phoneCodeHash;
};

const onSubmit = async () => {
  // const session = new StringSession("");

  // const client = new TelegramClient(session, +apiId.value, apiHash.value, {});

  // const code = await client.sendCode(
  //   { apiId: +apiId.value, apiHash: apiHash.value },
  //   phoneNumber.value
  // );

  // console.log(code);

  // await client.start({
  //   phoneNumber: async () => await Promise.reject(phoneNumber.value),
  //   phoneCode: async () => await Promise.resolve(phoneCode.value),
  //   onError: (err) => console.log(err),
  // });

  const res = await client.invoke(
    new Api.auth.SignIn({
      phoneNumber: phoneNumber.value,
      phoneCode: phoneCode.value,
      phoneCodeHash: phoneCodeHash.value,
    })
  );

  client.session.processEntities(res);

  await client.sendMessage("me", { message: "You're successfully logged in!" });

  client.addEventHandler(func);
};

// onMounted(async () => {
//   const session = new StringSession("");

//   const apiId = 24324859;
//   const apiHash = "2fed082b57efaa74e4c7466783b10eef";

//   const client = new TelegramClient(session, apiId, apiHash, {});

//   await client.connect();
// });

onBeforeUnmount(() => {
  console.log(client.listEventHandlers());

  client._eventBuilders = [];
  client.disconnect()
  // client.removeEventHandler(func,);
});
</script>
