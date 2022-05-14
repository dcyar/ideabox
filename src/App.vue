<template>
  <div class="container mx-auto p-4">
    <Teleport to="body">
      <RemoveIdea
        v-if="isModalActive"
        :name="ideaToRemove.name"
        @remove-cancel="isModalActive = !isModalActive"
        @remove-ok="removeIdea"
      />
    </Teleport>
    <div class="w-full bg-gray-100 shadow-lg p-4 rounded-lg">
      <h1 class="mb-5 text-4xl text-center">IdeaBox</h1>

      <AddIdea
        :user="user"
        @do-login="doLogin"
        @do-logout="doLogout"
        @add-idea="addIdea"
      />

      <TransitionGroup name="list">
        <AppIdea
          v-for="idea in ideas"
          :key="idea.createdAt"
          :user="user"
          :idea="idea"
          @vote-idea="voteIdea"
          @remove-idea="showRemoveIdeaModal"
          class="idea"
        />
      </TransitionGroup>
    </div>
  </div>
</template>

<script>
import { defineAsyncComponent, ref } from "vue";
import { GoogleAuthProvider, signInWithPopup } from "firebase/auth";
import {
  collection,
  addDoc,
  onSnapshot,
  doc,
  updateDoc,
  increment,
  query,
  orderBy,
  arrayUnion,
  setDoc,
  getDoc,
  deleteDoc,
} from "firebase/firestore";
import { auth, db } from "@/firebase.js";
import AppIdea from "@/components/AppIdea.vue";
import AddIdea from "./components/AddIdea.vue";

const RemoveIdea = defineAsyncComponent(() =>
  import("@/components/RemoveIdea.vue")
);

export default {
  name: "App",
  components: {
    AppIdea,
    AddIdea,
    RemoveIdea,
  },
  setup() {
    let user = ref(null);

    auth.onAuthStateChanged(async (auth) => {
      let userVotes;

      if (auth) {
        user.value = auth;
        userVotes = onSnapshot(doc(db, "votes", user.value.uid), (doc) => {
          if (doc.exists()) {
            let document = doc.data();
            if (document && "ideas" in document) {
              user.value.votes = document.ideas;
            }
          }
        });
      } else {
        user.value = null;
        userVotes && userVotes();
      }
    });

    const doLogin = async () => {
      const provider = new GoogleAuthProvider();

      try {
        await signInWithPopup(auth, provider);
      } catch (err) {
        console.log(err);
      }
    };

    const doLogout = async () => {
      try {
        await auth.signOut();
      } catch (err) {
        console.log(err);
      }
    };

    const ideas = ref([]);
    let isModalActive = ref(false);
    let ideaToRemove = {};

    onSnapshot(
      query(collection(db, "ideas"), orderBy("votes", "desc")),
      (snapshot) => {
        const newIdeas = [];

        snapshot.forEach((doc) => {
          const { name, user, userName, votes, createdAt } = doc.data();

          newIdeas.push({ id: doc.id, name, user, userName, votes, createdAt });
        });

        ideas.value = newIdeas;
      },
      (error) => console.log(error)
    );

    const addIdea = async (data) => {
      try {
        await addDoc(collection(db, "ideas"), {
          name: data.value,
          user: user.value.uid,
          userName: user.value.displayName,
          votes: 0,
          createdAt: Date.now(),
        });
      } catch (err) {
        console.log(err);
      }
    };

    const voteIdea = async ({ id, type }) => {
      try {
        let votes = await getDoc(doc(db, "votes", user.value.uid));

        if (votes.exists()) {
          votes = votes.data().ideas;

          if (votes.find((vote) => vote === id))
            throw new Error("User already voted!");
        }

        await updateDoc(doc(db, "ideas", id), {
          votes: increment(type ? 1 : -1),
        });

        await setDoc(
          doc(db, "votes", user.value.uid),
          {
            ideas: arrayUnion(id),
          },
          { merge: true }
        );
      } catch (err) {
        console.log(err);
      }
    };

    const showRemoveIdeaModal = ({ name, id }) => {
      ideaToRemove.name = name;
      ideaToRemove.id = id;
      isModalActive.value = true;
    };

    const removeIdea = async () => {
      try {
        await deleteDoc(doc(db, "ideas", ideaToRemove.id));
        ideaToRemove = {};
        isModalActive.value = false;
      } catch (err) {
        console.log(err);
      }
    };

    return {
      ideas,
      user,
      doLogin,
      doLogout,
      addIdea,
      voteIdea,
      isModalActive,
      ideaToRemove,
      showRemoveIdeaModal,
      removeIdea,
    };
  },
};
</script>

<style>
.list-move,
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}

.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

.list-leave-active {
  position: absolute;
}

.idea {
  @apply bg-gray-200;
}

.idea:nth-of-type(1) {
  @apply bg-red-500;
}

.idea:nth-of-type(2) {
  @apply bg-red-400;
}

.idea:nth-of-type(3) {
  @apply bg-red-300;
}

.idea:nth-of-type(4) {
  @apply bg-red-200;
}

.idea:nth-of-type(5) {
  @apply bg-red-100;
}

.user-actions {
  @apply mt-2 text-center;
}

.user-actions a {
  font-weight: bold;
  text-decoration: underline;
}
</style>
