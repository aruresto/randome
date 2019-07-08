<template>
  <el-container>
    <el-main>
      <el-row type="flex">
        <el-col :md="12" class="result-col">
          <result
            :disabled-random="!canRandom"
            :item="resultShowing"
            :latest-reward="latestReward"
            :random-handler="randomItemsHandler"
            :current-reward="currentReward"
          />
        </el-col>
        <el-col :md="12">
          <el-row>
            <el-col :span="12">
              <el-button type="warning" @click="rewardAddHandler">เพิ่มรางวัล</el-button>
            </el-col>
            <el-col :span="12">
              <el-button type="primary" @click="inputFileHandler">{{ inputBtnText }}</el-button>
              <input type="file" id="file" hidden @change="readFileHandler" />
            </el-col>
          </el-row>
          <el-row>
            <el-col class="text-left">
              <label>รางวัล</label>
              <reward-table :inputs="rewards" />
            </el-col>
          </el-row>
          <el-row>
            <el-col class="text-left">
              <label>ข้อมูลสุ่ม</label>
              <input-table :columns="colInputs" :inputs="inputs" />
            </el-col>
          </el-row>
          <!-- <el-row>
            <el-col>
              <el-button
                type="danger"
                :disabled="isRandoming || rewards.length === 0"
                @click="randomItemsHandler"
                autofocus
                v-show="!noInput"
              >สุ่ม !</el-button>
            </el-col>
          </el-row>-->
        </el-col>
      </el-row>
      <el-row>
        <el-col class="text-left">
          <label>ผลลัพธ์</label>
          <result-table :raw-columns="cols" :columns="colInputs" :items="results" />
        </el-col>
      </el-row>
    </el-main>
  </el-container>
</template>

<script>
import InputTable from "./InputTable";
import Result from "./Result";
import ResultTable from "./ResultTable";
import RewardTable from "./RewardTable";
import { setTimeout } from "timers";
import XLSX from "xlsx";
import Swal from "sweetalert2";

export default {
  name: "input-component",
  components: {
    InputTable,
    Result,
    ResultTable,
    RewardTable
  },
  data() {
    return {
      inputs: [],
      colInputs: [],
      cols: [],
      results: [],
      rewards: [],
      specificCol: [],
      currentReward: null,
      resultShowing: "",
      isRandoming: false
    };
  },
  computed: {
    noInput() {
      return this.inputs.length === 0;
    },
    inputBtnText() {
      return this.noInput ? "นำข้อมูลเข้า" : "นำข้อมูลเข้าเพิ่ม";
    },
    rewardCount() {
      return this.rewards.length + 1;
    },
    canRandom() {
      return !this.noInput && !this.isRandoming && this.rewards.length > 0;
    },
    latestReward() {
      if (this.rewards.length > 0) {
        let latestRewardIndex = this.rewards.length - 1;
        return {
          rewardNumber: latestRewardIndex + 1,
          reward: this.rewards[latestRewardIndex]
        };
      } else {
        return null;
      }
    }
  },
  mounted() {
    this.specificCol = process.env.VUE_APP_COLUMNS_SHOW.split(",") || [];
  },
  methods: {
    readFileHandler(el) {
      let file = el.target.files[0];

      /* Boilerplate to set up FileReader */
      const reader = new FileReader();
      reader.onload = e => {
        /* Parse data */
        const bstr = e.target.result;
        const wb = XLSX.read(bstr, { type: "binary" });
        /* Get first worksheet */
        const wsname = wb.SheetNames[0];
        const ws = wb.Sheets[wsname];
        /* Convert array of arrays */
        const data = XLSX.utils.sheet_to_json(ws, { header: 1 });
        /* Update state */
        this.inputs = [
          ...this.inputs,
          ...data.filter((row, index) => index !== 0)
        ];

        if (this.colInputs.length > 0) {
          let columnRow = data.find((row, index) => index === 0);

          this.colInputs = columnRow.map((column, index) => ({
            index,
            name: column
          }));

          if (this.specificCol.length >= 1) {
            this.colInputs = this.colInputs.filter(column =>
              this.specificCol.includes(column.name)
            );
          }
        }
        let columnRow = data.find((row, index) => index === 0);

        this.colInputs = columnRow.map((column, index) => ({
          index,
          name: column
        }));

        this.cols = this.colInputs;

        if (this.specificCol.length >= 1) {
          this.colInputs = this.colInputs.filter(column =>
            this.specificCol.includes(column.name)
          );
        }
        document.getElementById("file").value = "";
      };
      reader.readAsBinaryString(file);
    },
    findColumnIndex(columnName) {
      return this.colInputs.find(column => column.name === columnName);
    },
    inputFileHandler() {
      let file = document.getElementById("file");
      file.click();
    },
    async randomItemsHandler() {
      this.isRandoming = true;
      this.currentReward = this.latestReward;
      let randomIndex = Math.floor(Math.random() * this.inputs.length);
      let resultItem = this.inputs[randomIndex];
      if (this.inputs.length === 1) {
        this.results = [...this.results, this.inputs[0]];
        this.inputs = [];
        this.resultShowing =
          resultItem[
            this.findColumnIndex(process.env.VUE_APP_COLUMS_RESULT_SHOW).index
          ];
      } else {
        let randomTrigger = true;
        let randomShowingIndex = 0;
        let endIndex = this.inputs.length - 1;

        setTimeout(() => {
          randomTrigger = false;
        }, 3000);

        while (randomTrigger) {
          this.resultShowing = this.inputs[randomShowingIndex][
            this.findColumnIndex(process.env.VUE_APP_COLUMS_RESULT_SHOW).index
          ];

          if (randomShowingIndex === endIndex) {
            randomShowingIndex = 0;
          } else {
            randomShowingIndex++;
          }

          await this.sleep(50);
        }

        this.resultShowing =
          resultItem[
            this.findColumnIndex(process.env.VUE_APP_COLUMS_RESULT_SHOW).index
          ];

        let newResult = [
          ...this.inputs[randomIndex],
          this.latestReward.reward,
          this.latestReward.rewardNumber
        ];

        this.results = [...this.results, newResult];
        this.inputs = this.inputs.filter(
          (input, index) => index !== randomIndex
        );
        this.rewards = this.rewards.filter(
          (reward, index) => index !== this.rewards.length - 1
        );
      }

      this.isRandoming = false;
    },
    sleep(time) {
      return new Promise(resolve => setTimeout(resolve, time));
    },
    async rewardAddHandler() {
      const { value: reward } = await Swal.fire({
        title: `เพิ่มรางวัลที่ ${this.rewardCount}`,
        input: "text",
        showCancelButton: true,
        confirmButtonText: "เพิ่ม",
        cancelButtonText: "ยกเลิก",
        inputValidator: value => {
          if (!value) {
            return "ข้อความว่างเปล่า กรุณากรอกชื่อรางวัล";
          }
        }
      });

      if (reward) {
        this.rewards = [...this.rewards, reward];
      }
    }
  }
};
</script>

<style lang="scss" scoped>
.text-left {
  text-align: left;
}
.result-col {
  display: flex;
  align-items: center;
}
</style>
