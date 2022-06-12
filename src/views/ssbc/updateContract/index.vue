<template>
  <div class="app-container">
    <el-row :gutter="1">
      <el-col :span="14" :offset="1" :justify="center">
        <el-alert
          title="智能合约编辑与上传"
          type="info"
          description="请使用Go语言编写"
          show-icon
        />
      </el-col>
    </el-row>
    <br>
    <el-form>
      <el-form-item></el-form-item>
      <el-form-item></el-form-item>
      <el-form-item></el-form-item>
      <el-form-item label="发起地址">
        <el-col :span="12">
          <el-select v-model="form.account" style="width: 100%" class="filter-item">
            <el-option v-for="user in userList" :key="user" :label="user.address" :value="user.address" @click.native="chooseSender(user)" />
          </el-select>
        </el-col>
      </el-form-item>
      <el-form-item label="私钥">
        <el-col :span="12">
          <el-input v-model="form.private_key" :disabled="true" />
        </el-col>
      </el-form-item>
      <el-form-item label="公钥">
        <el-col :span="12">
          <el-input v-model="form.public_key" :disabled="true" />
        </el-col>
      </el-form-item>
      <el-form-item label="合约名称">
        <el-col :span="12">
          <el-input v-model="form.name" />
        </el-col>
      </el-form-item>
      <el-form-item label="编辑合约" />
      <el-col :span="12">
        <codemirror v-model="form.code" :options="cmOption" />
      </el-col>
      <el-form-item />
      <el-button type="primary" :disabled="disable" style="alignment: center; width:30%" @click="postContract">创建合约</el-button>
    </el-form>
  </div>
</template>

<script>
import dedent from 'dedent'
import { codemirror } from 'vue-codemirror'
// language
// import 'codemirror/mode/javascript/javascript.js'
import 'codemirror/mode/go/go.js'
// theme css
import 'codemirror/theme/monokai.css'

import Cookies from 'js-cookie'
import { query, postContract } from '@/api/ssbc'

export default {
  components: {
    codemirror
  },
  data() {
    return {
      q: {
        type: 'getAllAccounts',
        parameters: []
      },
      userList: [],
      form: {
        account: '',
        private_key: '',
        public_key: '',
        name: '',
        type: 2,
        code: dedent`
package main

import (
	"math/rand"
	"time"
)

func random(args map[string]string) (interface{}, error) {
	rand.Seed(time.Now().Unix())
	return rand.Intn(100), nil
}
        `
      },
      user: {},
      cmOption: {
        tabSize: 4,
        lineNumbers: true,
        line: true,
        mode: 'text/x-go',
        matchBrackets: true,
        theme: 'monokai'
      },
      disable: false
    }
  },
  created() {
    this.getAllAccounts()
  },
  methods: {
    postContract() {
      this.disable = true
      setTimeout(() => {
        this.disable = false
      }, 4000)
      postContract(this.form).then(res => {
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
          this.form.account = ''
          this.form.private_key = ''
          this.form.public_key = ''
          Cookies.set('PublicKey', '')
          Cookies.set('PrivateKey', '')
          Cookies.set('AccountAddress', '')
        } else {
          this.form.account = Cookies.get('AccountAddress')
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
    }
  }
}
</script>
