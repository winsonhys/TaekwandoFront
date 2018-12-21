<template>
  <div>
    <div id="paymentAndAttendanceHeader">
      <span>Payment History</span>
    </div>
    <el-row id="commentsRow">
      <el-row>
        <date-selector style="margin-left: 20px;" v-model="paymentDateRange"></date-selector>
      </el-row>
      <el-table max-height="500" :data="paymentData" style="width: 90%">
        <el-table-column prop="mode" label="Payment mode"></el-table-column>
        <el-table-column prop="type" label="Item paid for"></el-table-column>
        <el-table-column prop="created" label="Paid on"></el-table-column>
        <el-table-column prop="description" label="Payment Description"></el-table-column>
        <el-table-column prop="price" label="Amount"></el-table-column>
        <el-table-column label="Operations" fixed="right">
          <template slot-scope="scope">
            <el-button @click="generateReceipt(scope.row)" type="text" size="small">Download Receipt</el-button>
            <el-button
              v-if="scope.row.mode !== REFUNDED"
              @click="refund(scope.row)"
              type="text"
              size="small"
            >Refund</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-row>
  </div>
</template>

<script>
import moment from "moment"
import RecieptGenerator from "@/assets/reciept"
import DateSelector from "@/components/utils/DateSelector"

export default {
  components: {
    DateSelector,
  },
  computed: {
    paymentData() {
      const details = this.$store.getters.getStudentInfo(this.userId)
      let filteredData = _.map(details.payments, (values, id) => ({
        ...values,
        id,
      }))

      if (
        this.paymentDateRange.length > 0 &&
        !moment(this.paymentDateRange[0]).isSame(moment(0))
      ) {
        const startDate = this.paymentDateRange[0]
        const endDate = this.paymentDateRange[1]
        const selectionRange = moment.range(startDate, endDate)
        filteredData = _.filter(filteredData, (data) => {
          return selectionRange.contains(moment(data.created))
        })
      }
      const displayData = _.map(filteredData, (data) => {
        const paymentType = data.type
        let parsedType = ""
        for (const type of paymentType) {
          parsedType = parsedType + `${type} / `
        }
        return {
          ...data,
          type: parsedType,
          created: moment(data.created).format("DD-MM-YYYY"),
        }
      })
      return displayData
    },
  },
  data() {
    return {
      paymentDateRange: "",
    }
  },
  methods: {
    refund(rowData) {
      this.$store.dispatch("refundPayment", {
        ...rowData,
        userId: this.userId,
        vm: this,
      })
    },
    generateReceipt({ type, description, price }) {
      const paymentType = type.split(" / ")
      RecieptGenerator(paymentType, description, price)
    },
  },
  props: {
    userId: {
      type: String,
      required: true,
    },
  },
}
</script>

<style scoped>
#paymentAndAttendanceHeader {
  font-size: 25px;
}
#commentsRow {
  margin-bottom: 20px;
  margin-top: 20px;
}
</style>