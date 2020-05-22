<template>
  <div class="container">
    <h1 class="title" v-text="title" />
    <div class="card">
      <p class="image-area">
        <img :src="product.image" :alt="product.name" class="image" />
      </p>
      <div class="info-area">
        <h2 class="name" v-text="product.name" />
        <p class="desc" v-text="product.desc" />
        <p
          class="amount"
          v-text="'¥' + product.amount.toLocaleString() + '-'"
        />
        <client-only>
          <p class="stripe-area">
            <card
              :options="stripeOptions"
              :stripe="stripePK"
              class="stripe"
              @change="isEntered = $event.complete"
            />
          </p>
        </client-only>
        <div class="message" v-text="message" />
        <p class="button-area">
          <button
            class="button"
            :class="{ active: isEntered }"
            aria-label="決済する"
            @click="pay"
            v-text="'決済する'"
          />
        </p>
      </div>
      <div class="complete" :class="{ active: isComplete }">
        <p class="message" v-text="'Thank you!'" />
      </div>
    </div>
  </div>
</template>
<script>
import axios from 'axios'
import { Card, createToken } from 'vue-stripe-elements-plus'
export default {
  components: {
    Card
  },
  data() {
    return {
      title: '決済フォーム',
      product: {
        name: 'サンプル腕時計',
        desc:
          'こちらは決済フォームのサンプルのためご購入はできません。また、カード番号を入力しても請求されることはありません。ご理解いただいた上でお進みください。カード番号は「4242 4242 4242 4242」をご入力ください。※年月,CVCは任意の値で結構です。',
        amount: 12345,
        image: 'watch.jpg'
      },
      stripeOptions: { hidePostalCode: true },
      stripePK: 'pk_test_87vXgp7lt7eQx9IsTAPG6SVw00SkJVRHj7',
      message: '',
      isEntered: false,
      isComplete: false
    }
  },
  methods: {
    async pay() {
      try {
        // 決済用トークン発行
        const tokenResult = await createToken()
        if (
          !tokenResult ||
          !tokenResult.token ||
          !tokenResult.token.id ||
          tokenResult.token.id === ''
        ) {
          throw new Error('トークン発行エラー')
        }

        // 決済処理
        const chargeResult = await axios.post(
          `${process.env.FUNCTION_URL}/.netlify/functions/charge`,
          {
            amount: this.product.amount,
            token: tokenResult.token.id
          }
        )
        if (!chargeResult || chargeResult.data !== 'NORMAL') {
          throw new Error('決済エラー')
        }
        this.isComplete = true
      } catch (error) {
        this.message = error.message + 'が発生しました。'
      }
    }
  },
  head() {
    return {
      title: this.title,
      script: [{ src: '//js.stripe.com/v3/' }]
    }
  }
}
</script>
<style lang="scss" scoped>
.container {
  padding: 24px;

  .title {
    text-align: center;
    font-size: 30px;
  }

  .card {
    border-radius: 8px;
    max-width: 480px;
    margin: 0 auto;
    margin-top: 20px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
    position: relative;

    p {
      margin-top: 12px;
    }

    .image-area {
      border-radius: 4px;
      overflow: hidden;
      .image {
        width: 100%;
        height: 240px;
        object-fit: cover;
      }
    }

    .info-area {
      padding: 18px;
      .name {
        font-size: 26px;
        text-align: center;
      }

      .amount {
        font-size: 24px;
        text-align: right;
      }

      .stripe-area {
        .stripe {
          border-bottom: dashed 1px lightgrey;
        }
      }

      .message {
        color: red;
        text-align: center;
      }

      .button-area {
        text-align: center;

        .button {
          cursor: pointer;
          background-color: lightgray;
          color: white;
          border: 0;
          border-radius: 4px;
          box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
          padding: 4px 16px;
          font-size: 18px;

          &.active {
            background-color: orange;
          }
        }
      }
    }

    .complete {
      position: absolute;
      top: 0;
      bottom: 0;
      left: 0;
      right: 0;
      width: 100%;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
      background-color: rgba(255, 255, 255, 0.9);
      opacity: 0;
      z-index: -1;

      &.active {
        opacity: 1;
        z-index: 5;
      }

      .message {
        display: inline-flex;
        align-items: center;
        justify-content: center;
        color: red;
        border: solid 2px red;
        border-radius: 25.5px;
        font-size: 36px;
        padding: 4px 24px;
        transform: rotate(-15deg);
      }
    }
  }
}
</style>
