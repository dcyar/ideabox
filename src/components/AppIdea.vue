<template>
  <article class="p-3 mb-3 rounded-lg sm:flex sm:items-center">
    <img
      v-if="userIdea"
      @click="removeIdea"
      class="mr-3 cursor-pointer"
      src="@/assets/img/remove.svg"
      alt="remove idea"
    />
    <section class="text-center sm:flex-1 sm:text-left">
      <h2 class="text-xl sm:text-2xl sm:leading-6">
        {{ idea.name }}
      </h2>
      <small>{{ idea.userName }}</small>
    </section>
    <section
      class="pt-3 border-t-2 mt-6 border-black sm:pt-0 sm:pl-3 sm:border-t-0 sm:border-l-2 sm:mt-0 sm:flex sm:items-center"
    >
      <h3 class="text-3xl font-bold text-center">{{ idea.votes }}</h3>
      <nav
        v-if="user && !userVoted"
        class="flex justify-center sm:block sm:ml-2"
      >
        <img
          @click="voteIdea(true)"
          class="w-10 cursor-pointer"
          src="@/assets/img/arrow.svg"
          alt="Vote up"
        />
        <img
          @click="voteIdea(false)"
          class="w-10 cursor-pointer transform rotate-180"
          src="@/assets/img/arrow.svg"
          alt="Vote down"
        />
      </nav>
    </section>
  </article>
</template>
<script>
import { computed } from "@vue/runtime-core";
export default {
  name: "AppIdea",
  props: {
    user: {
      type: [Object, null],
    },
    idea: {
      type: Object,
      required: true,
    },
  },
  emits: ["vote-idea", "remove-idea"],
  setup(props, { emit }) {
    const voteIdea = (type) => emit("vote-idea", { id: props.idea.id, type });

    const userIdea = computed(
      () => props.user && props.user.uid === props.idea.user
    );

    const userVoted = computed(() => {
      if (props.user.votes) {
        return props.user.votes.find((item) => item === props.idea.id);
      }
    });

    const removeIdea = () =>
      emit("remove-idea", { name: props.idea.name, id: props.idea.id });

    return { voteIdea, userVoted, removeIdea, userIdea };
  },
};
</script>
