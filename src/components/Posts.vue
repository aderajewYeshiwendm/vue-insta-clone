<script setup>
import { DotsHorizontalIcon } from '@heroicons/vue/solid';
import { HeartIcon, ChatIcon, PaperAirplaneIcon, BookmarkIcon, EmojiHappyIcon } from '@heroicons/vue/outline';
import { HeartIcon as HeartIconSolid } from "@heroicons/vue/solid";
import { collection, addDoc,setDoc, doc, serverTimestamp, orderBy, onSnapshot, query } from "firebase/firestore";
import { db } from "../firebaseConfig";
import { auth } from "../firebaseConfig";
import { ref, watchEffect } from "vue";
import Comments from "../components/Comments.vue";
const comment = ref("");
const comments = ref([]);
const isLiked = ref(false);
const likes = ref([]);
const props = defineProps({
    id: {
        type: String,
        required: true,
    },
    username: {
        type: String,
        required: true,
    },
    profile: {
        type: String,
        required: true,
    },
    text: {
        type: String,
        required: true,
    },
    media: {
        type: String,
    },
    timeStamp: {
        type: Object
    }
});
const postComments = async () => {
    await addDoc(collection(db, "posts", props.id, "comments"), {
        id: auth.currentUser?.uid,
        username: auth?.currentUser?.displayName,
        profile: auth?.currentUser?.photoURL,
        comment: comment.value,
        timeStamp: serverTimestamp()
    });
    setTimeout(() => {
        comment.value = "";
    }, 1000)
};
// comments
watchEffect(() => {
    const commentRef = collection(db, "posts", props.id, "comments");
    const q = query(commentRef, orderBy("timeStamp", "desc"));
    const unsubscribe = onSnapshot(q, querySnapshot => {
        let commentsData = [];
        querySnapshot.forEach(doc => {
            commentsData.push({...doc.data(), id: doc.id})
        });
        comments.value = commentsData;
    });
    return () => unsubscribe();
});
watchEffect(() => {
    const likesRef = collection(db, "posts", props.id, "likes");
    const q = query(likesRef);
    const unsubscribe = onSnapshot(q, querySnapshot => {
        let postLikes = [];
        querySnapshot.forEach(doc => {
            postLikes.push({...doc.data(), id: doc.id});
        });
        likes.value = postLikes;
    });
    return () => unsubscribe();
});
watchEffect(() => {
      isLiked.value = likes.value.findIndex((post) => post.id === auth.currentUser.uid ) !== -1;
})
// like functionality
const likePost = async () => {
    await setDoc(doc(db, "posts", props.id, "likes", auth?.currentUser?.uid), {
            username: props.username
    })
};
</script>
<template>
<main>
    <div class="bg-white my-6 border border-gray-200 rounded-md">
        <div class="flex justify-between items-center p-2 border-b border-gray-200">
            <div class="flex items-center">
                <img :src="props.profile" alt="Image" class="h-10 w-10 rounded-full">
                <h4 class="font-bold pl-3">{{ props.username }}</h4>
            </div>
            <DotsHorizontalIcon class="h-6 w-6 text-gray-500"/>
        </div>
        <div>
            <img :src="props.media" :alt="props.username" class="w-full object-cover">
        </div>
        <div class="flex justify-between pt-3 px-3">
            <div class="flex space-x-4">
                <div>
                    <div v-if="isLiked" class="text-red-500" @click="likePost()"><HeartIconSolid class="icon-style"/></div>
                    <div v-else  @click="likePost()"><HeartIcon class="icon-style"/></div>
                </div>
                <div>
                    <ChatIcon class="icon-style"/>
                </div>
                <div>
                    <PaperAirplaneIcon class="icon-style"/>
                </div>
            </div>
            <div>
                <BookmarkIcon class="icon-style"/>
            </div>
        </div>
        <div>
            <div class="px-4 py-2 truncate">
                <div v-if="likes.length > 0">
                    <p class="font-bold">{{likes.length}} Likes</p>
                </div>
                <div class="flex items-center mt-2">
                        <p class="mr-3 font-bold">{{props.username}}</p>
                     <p class="truncate">{{props.text}}</p>
                </div>
            </div>
        </div>
        <!-- comments -->
        <div class="overflow-y-scroll max-h-20 mb-4 ml-4 scrollbar-thin">
            <div class="items-center mb-2 space-x-2" v-for="comment in comments" :key="comment.id">
            <Comments
            :id="comment.id"
            :profile="comment.profile"
            :username="comment.username"
            :comment="comment.comment"
            :time="comment.timeStamp"
             />
            </div>
        </div>
        <div class="ml-4 text-gray-400 text-xs lg:text-base font-bold">
            <timeago :datetime="props?.timeStamp?.toDate()" :auto-update="60"></timeago>
        </div>
        <!-- comment input -->
        <div class="flex justify-between items-center p-2">
            <div class="flex items-center space-x-2">
               <div>
                 <EmojiHappyIcon class="w-7 h-7"/>
               </div>
                <div>
                    <input type="text" placeholder="Add a comment.." class="py-2 px-2 w-full appearance-none focus:outline-none" v-model="comment">
                </div>
            </div>
            <div>
                 <button :disabled="!comment" class="bg-[#1d9bf0] text-white rounded-full shadow-md hover:bg-[#1a8cd8] disabled:hover:bg-[#1d9bf0] disabled:opacity-50 disabled:cursor-default py-1.5 px-4 font-bold" @click="postComments()">Post</button>
            </div>
        </div>
        </div>
</main>
</template>