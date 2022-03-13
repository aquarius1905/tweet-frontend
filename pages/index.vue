<template>
<div class="tweet">
  <div class="tweet_wrapper">
    <h1 class="ttl">独り言APP</h1>
    <validation-observer ref="obs" tag="div">
      <validation-provider mode="passive" rules="required" v-slot="{ errors }" >
        <label for="tweet_content">つぶやく内容を入力してください</label>
        <br/>
        <input type="text" name="内容" class="tweet_content" v-model="tweet_content" />
        <button @click="addTweet">つぶやく</button>
        <div class="error">{{ errors[0] }}</div>
      </validation-provider>
    </validation-observer>
    <h2 class="tweet_list_ttl">つぶやき一覧</h2>
    <div class="tweet_list">
      <div v-for="(item, index) in tweet_items" :key="index" class="tweet_item">
        <div class="tweet_item_wrapper">
        <div class="circle"></div>
        <div class="tweet">{{ item.tweet }}</div>
      </div>
      <button @click="deleteTweet(item.id, index)" class="delete_btn">削除</button>
      </div>
    </div>
  </div>
</div>
</template>

<script>
export default {
  data() {
    return {
      tweet_content: '',
      tweet_items: []
    }
  },
  methods: {
    async getTweets() {
      this.tweet_items.splice(0);
      const {data} = await this.$axios.get("/api/v1/tweet/");
      for(const tweetData of data.data) {
        const tweetItem = { 
          id: tweetData.id, 
          tweet: tweetData.tweet
        };
        this.tweet_items.unshift(tweetItem);
      }
    },
    async addTweet() {
      const isValid = await this.$refs.obs.validate();
      if(!isValid) return;
      const sendData = { tweet: this.tweet_content }
      console.log(sendData);
      const {data} = await this.$axios.post("/api/v1/tweet/", sendData);
      const tweetItem = {
        id: data.data.id,
        tweet: data.data.tweet
      };
      this.tweet_items.unshift(tweetItem);
      this.tweet_content = '';
    },
    async deleteTweet(id, index) {
      await this.$axios.delete("/api/v1/tweet/" + id);
      this.tweet_items.splice(index, 1);
    }
  },
  created() {
    this.getTweets();
  }
}
</script>

<style scoped>
.tweet_wrapper {
  margin: 0 auto;
  text-align: center;
}
.ttl {
  margin-bottom: 50px;
}
.tweet_list_ttl {
  margin-top: 60px;
}
.tweet_list {
  width: 45%;
  margin: 30px auto 0;
  border: 5px solid #ddd;
  padding: 20px;
}
.tweet {
  font-size: 20px;
  align-items: center;
}
.circle {
    display: inline-block;
    width: 50px;
    height: 50px;
    border-radius: 50%;
    background-color: #f00;
    margin: 0 10px 6px 0;
}
.tweet_item {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}
.tweet_item_wrapper {
  display: flex;
  align-items: center;
}
.delete_btn {
  margin-left: auto;
  width: 50px;
  height: 30px;
}
.tweet_content {
  font-size: 16px
}
.error {
  font-size: 16px;
  font-weight: bold;
  color: #f00;
}
</style>
