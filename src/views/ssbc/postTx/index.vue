<template>
  <div class="app-container">
    <el-form>
      <el-form-item></el-form-item>
      <el-form-item></el-form-item>
      <el-form-item></el-form-item>
      <el-form-item label="发起地址" label-width="25%">
        <el-col :span="14">
          <el-select v-model="form.from" style="width: 100%" class="filter-item" filterable>
            <el-option v-for="user in userList" :key="user.address" :label="user.address" :value="user.address" @click.native="chooseSender(user)" />
          </el-select>
        </el-col>
      </el-form-item>

      <el-form-item label="私钥" label-width="25%">
        <el-col :span="14">
          <el-input v-model="form.private_key" :disabled="true" />
        </el-col>
      </el-form-item>
      <el-form-item label="公钥" label-width="25%">
        <el-col :span="14">
          <el-input v-model="form.public_key" :disabled="true" />
        </el-col>
      </el-form-item>
      <el-form-item label="接收地址" label-width="25%">
        <el-col :span="14">
          <el-select v-model="form.to" style="width: 100%" filterable>
            <el-option v-for="user in userList" :key="user.address" :label="user.address" :value="user.address" @click.native="chooseReceiver(user)" />
          </el-select>
        </el-col>
      </el-form-item>
      <el-form-item label="金额" label-width="25%">
        <el-col :span="14">
          <el-input v-model.number="form.value" maxlength="10"/>
        </el-col>
      </el-form-item>
      <el-form-item>
        <el-col :span="14" :offset="9">
          <el-button type="primary" :disabled="disable" style="alignment: center;width:30% " @click="postTran">发起转账</el-button>
        </el-col>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import Cookies from 'js-cookie'
import { query, postTran } from '@/api/ssbc'

export default {
  data() {
    return {
      q: {
        type: 'getAllAccounts',
        parameters: []
      },
      userList: [],
      form: {
        private_key: '',
        public_key: '',
        from: '',
        to: '',
        contract: '',
        value: 100,
        method: '',
        dest: '',
        args: '{}',
        type: 0
      },
      disable: false
    }
  },
  created() {
    this.getAllAccounts()
  },
  methods: {
    postTran() {
      this.disable = true
      setTimeout(() => {
        this.disable = false
      }, 1000)
      postTran(this.form).then(res => {
        if (res.error === '') {
          this.$message({
            message: '成功提交',
            type: 'success'
          })
        } else {
          this.$message({
            message: res.error,
            type: 'warning'
          })
        }
      })
    },
    getAllAccounts() {
      query(this.q).then(res => {
        // 筛选出普通账户和智能合约账户
        const user = []
        const contract = []
        const total = res.data
        total.forEach(function(r) {
          if (!r.iscontract) {
            user.push(r)
          } else {
            contract.push(r)
          }
        })
        this.userList = user
        if (user.length === 0) {
          this.form.from = ''
          this.form.private_key = ''
          this.form.public_key = ''
          Cookies.set('PublicKey', '')
          Cookies.set('PrivateKey', '')
          Cookies.set('AccountAddress', '')
        } else {
          this.form.from = Cookies.get('AccountAddress')
          this.form.private_key = Cookies.get('PrivateKey')
          this.form.public_key = Cookies.get('PublicKey')
        }
      })
    },
    chooseSender(item) {
      this.form.from = item.address
      this.form.public_key = item.publickey
      this.form.private_key = item.privatekey

      Cookies.set('AccountAddress', item.address)
      Cookies.set('PublicKey', item.publickey)
      Cookies.set('PrivateKey', item.privatekey)
    },
    chooseReceiver(item) {
      this.form.to = item.address
    }
  }
}
</script>
