<template>
  <section class="mb-6">
    <form @submit.prevent="addIdea" class="sm:flex">
      <input
        v-model="idea"
        type="text"
        class="w-full p-3 sm:flex-auto"
        required
        placeholder="Add your idea"
      />
      <input
        v-if="user"
        type="submit"
        class="w-full p-2 bg-gray-600 text-white sm:flex-1"
        value="Add idea"
      />
    </form>
    <p v-if="!user" class="user-actions">
      Please
      <a href="#" @click="doLogin">login</a>
      first
    </p>
    <p v-else class="user-actions">
      Hi {{ user.displayName }}, <a href="#" @click="doLogout">Logout</a>
    </p>
  </section>
</template>

<script>
import { ref } from "vue";

export default {
  name: "AddIdea",
  props: {
    user: {
      type: [Object, null],
    },
  },
  emits: ["do-login", "do-logout", "add-idea"],
  setup(props, { emit }) {
    const idea = ref("");

    const doLogin = () => emit("do-login");
    const doLogout = () => emit("do-logout");
    const addIdea = () => {
      emit("add-idea", idea);
      idea.value = "";
    };

    return { idea, doLogin, doLogout, addIdea };
  },
};
</script>
