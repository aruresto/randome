<template>
  <el-container>
    <el-main>
      <el-row>
        <el-col :span="24">
          <result
            :is-start="isStart"
            :disabled-random="!canRandom"
            :item="resultShowing"
            :latest-reward="latestReward"
            :random-handler="randomItemsHandler"
            :current-reward="currentReward"
          />
        </el-col>
        <el-col :span="24">
          <el-row v-show="isStart">
            <el-col>
              <el-button type="warning" @click="resetHandler">รีเซ็ต</el-button>
            </el-col>
          </el-row>
          <el-row v-show="!isStart">
            <el-col :span="8">
              <el-button type="danger" :disabled="!canRandom" @click="startRandom">เริ่ม !</el-button>
            </el-col>
            <el-col :span="8">
              <el-button type="warning" @click="rewardAddHandler">เพิ่มรางวัล</el-button>
            </el-col>
            <el-col :span="8">
              <el-button type="primary" @click="inputFileHandler">{{ inputBtnText }}</el-button>
              <input type="file" id="file" hidden @change="readFileHandler" />
            </el-col>
          </el-row>
          <el-row>
            <el-col class="text-left">
                <label>ผลลัพธ์</label>
          <result-table :raw-columns="cols" :columns="colInputs" :items="results" />
            
            </el-col>
          </el-row>
          <el-row>
            <el-col class="text-left">
                <label>รางวัล</label>
              <reward-table :inputs="rawRewards" />
             
            </el-col>
          </el-row>
        </el-col>
      </el-row>
      <el-row>
        <el-col class="text-left">
         <label>ข้อมูลสุ่ม</label>
              <input-table :columns="colInputs" :inputs="rawInputs" />
              <span>จำนวน {{ rawInputs.length }}</span>
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
      rawInputs: [],
      colInputs: [],
      cols: [],
      results: [],
      rewards: [],
      rawRewards: [],
      specificCol: [],
      currentReward: null,
      resultShowing: [],
      isRandoming: false,
      isStart: false
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
      this.specificCol = process.env.VUE_APP_COLUMNS_SHOW.split(",") || [];

      /* Boilerplate to set up FileReader */
      const reader = new FileReader();
      reader.onload = e => {
        /* Parse data */
        const bstr = e.target.result;
        const wb = XLSX.read(bstr, { type: "binary" });
        for (var i in wb.SheetNames) {
          /* Get first worksheet */
          const wsname = wb.SheetNames[i];
          const ws = wb.Sheets[wsname];
          if (!ws["!ref"]) {
            continue;
          }
          /* Convert array of arrays */
          let data = XLSX.utils.sheet_to_json(ws, { header: 1 });

          let columnRow = data
            .filter(row => row.length > 1)
            .find((row, index) => index === 0);
          let columnMap = [];

          let colInputs = columnRow.map((column, index) => ({
            index,
            name: column
          }));

          if (this.specificCol.length >= 1) {
            for (var iscol of this.specificCol) {
              var colSplit = iscol.split("|");
              for (var isubscol in colSplit) {
                let subCol = colSplit[isubscol];
                subCol = subCol.replace("(", "");
                subCol = subCol.replace(")", "");
                let filterInput = colInputs.find(
                  col => col.name.indexOf(subCol) > -1
                );
                if (filterInput) {
                  let colFind = this.colInputs.find(
                    selected => selected.rawName === iscol
                  );

                  if (colFind) {
                    let columnMapData;
                    if (columnMap[this.colInputs.indexOf(colFind)]) {
                      columnMapData =
                        columnMap[this.colInputs.indexOf(colFind)];
                    }
                    columnMap[this.colInputs.indexOf(colFind)] =
                      filterInput.index;

                    if (columnMapData) {
                      columnMap.push(columnMapData);
                    }
                  } else {
                    columnMap.push(filterInput.index);
                  }

                  if (!colFind) {
                    this.colInputs.push({
                      name: subCol,
                      rawName: iscol
                    });
                  }

                  break;
                }
              }
            }
          }

          data = data
            .filter(row => row.length > 1)
            .filter((row, index) => index !== 0);

          data = data.map(row => {
            let filterRow = [];

            for (var icolmap in columnMap) {
              filterRow.push(row[columnMap[icolmap]]);
            }
            return filterRow;
          });

          this.inputs = [...this.inputs, ...data];

          this.rawInputs = this.inputs;
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
        this.resultShowing = resultItem;
      } else {
        let randomTrigger = true;
        let randomShowingIndex = 0;
        let endIndex = this.inputs.length - 1;

        setTimeout(() => {
          randomTrigger = false;
        }, 3000);

        while (randomTrigger) {
          this.resultShowing = this.inputs[randomShowingIndex];

          if (randomShowingIndex === endIndex) {
            randomShowingIndex = 0;
          } else {
            randomShowingIndex++;
          }

          await this.sleep(50);
        }

        this.resultShowing = resultItem;

        let newResult = [
          this.latestReward.rewardNumber,
          this.latestReward.reward,
          ...this.inputs[randomIndex]
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
        this.rawRewards = this.rewards;
      }
    },
    startRandom() {
      this.isStart = true;
    },
    resetHandler() {
      this.inputs = [];
      this.rawInputs = [];
      this.colInputs = [];
      this.cols = [];
      this.results = [];
      this.rewards = [];
      this.rawRewards = [];
      this.specificCol = [];
      this.currentReward = null;
      this.resultShowing = "";
      this.isRandoming = false;
      this.isStart = false;
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
