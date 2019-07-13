<template>
  <el-container style="height: 100%;">
    <el-main style="height: 100%;">
      <el-row style="height:20%;">
        <el-col v-show="isStart">
          <h1 class="reward">{{ rewardText }}</h1>
        </el-col>
        <el-col>
          <el-button
            v-show="isStart"
            type="danger"
            :disabled="disabledRandom"
            @click="randomHandler"
            autofocus
          >สุ่ม !</el-button>
        </el-col>
      </el-row>
      <el-row style="height: 20%;">
        <el-col>
          <span>{{currentRewardText}}</span>
        </el-col>
      </el-row>
      <el-row style="height:50%;" type="flex" justify="center" align="center">
        <el-col>
          <el-row :key="index" v-for="(i, index) in item">
            <el-col>
              <span :class="{'result': index === 0}">{{ i }}</span>
            </el-col>
          </el-row>
        </el-col>
      </el-row>
    </el-main>
  </el-container>
</template>

<script>
export default {
  name: "result",
  props: {
    item: {
      type: Array,
      default: () => []
    },
    disabledRandom: {
      type: Boolean,
      default: false
    },
    latestReward: {
      type: Object,
      default: null
    },
    randomHandler: {
      type: Function,
      default: () => {
        return alert(`random handler`);
      }
    },
    currentReward: {
      type: Object,
      default: null
    },
    isStart: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    rewardText() {
      if (this.latestReward) {
        return `รางวัลต่อไป รางวัลที่ ${this.latestReward.rewardNumber}  - ${this.latestReward.reward} `;
      } else {
        return "";
      }
    },
    currentRewardText() {
      if (this.currentReward) {
        if (this.item !== "") {
          return ` รางวัลที่ ${this.currentReward.rewardNumber}  - ${this.currentReward.reward} `;
        } else {
          return "";
        }
      } else {
        return "";
      }
    }
  }
};
</script>

<style lang="scss" scoped>
.result {
  font-size: 5rem;
}

span {
  font-size: 2.5rem;
}
</style>
