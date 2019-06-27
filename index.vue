<!--
@description: 项目管理-评标-评标准备
  @author: XWl

  项目管理-评标-评标准备
  /bidding/contentNavProjectManger/evaluationBid/evaluationGetReady
-->
<template>
  <div>
    <div class="bIdStepBack_content" @click="movingPosition">
      <!--评标步骤-->
      <div class="page-title page-bg">
        01 评标步骤
        <el-button size="mini" style="margin-left: 10px;" @click="dEditStep">编辑步骤</el-button>
        <!--专家分组-->
        <el-dropdown trigger="click" placement="bottom">
          <span class="el-dropdown-link">
            <el-button size="mini" style="margin-left: 10px;">专家分组<i class="el-icon-arrow-down el-icon--right"/></el-button>
          </span>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item v-for="(item,index) in stepDatas" :key="index" @click.native="expertGroupingClick(item,index)">{{ item.name }}</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </div>
      <div class="evaluationSteps">
        <el-form ref="form" label-width="68px">
          <el-row>
            <el-col :span="24">
              <el-form-item ref="linker" label="所有步骤" prop="linker">
                <div class="allSteps">
                  <ul>
                    <li v-for="(item,index) in stepDatas" :key="index" @click="addStep(item,index)">
                      <i class=" iconfont icon-tianjia1 icon-add"/>
                      {{ item.name }}
                    </li>
                  </ul>
                </div>
              </el-form-item>
            </el-col>
            <el-col :span="24">
              <el-form-item ref="idCard" label="评标步骤" prop="idCard">
                <div class="step">
                  <ul ref="step" class="clearfix">
                    <!--已设置的步骤-->
                    <li v-for="(item,index) in step" :key="index" @click="editStepClick(item)" @contextmenu.prevent="menuClick($event,item,index)" @mousedown="mousedownStep($event,index,item)">
                      <i :class="item.icon"/>
                      {{ item.name }}
                      <span class="ping_x iconfont icon-guanbi" @click="deleteStep(item,index)"/>
                      <span class="ping_you iconfont icon-pingbiaoyoujiantou"/>
                    </li>
                  </ul>
                  <!--选择正在拖动的步骤-->
                  <p ref="downStep" class="downStep">
                    <i :class="downStep.icon"/>
                    {{ downStep.name }}
                  </p>
                  <!--菜单栏-->
                  <div id="menu_box">
                    <div ref="menu" class="menu">
                      <p @click="movingPosition('prev')">向前移动</p>
                      <p @click="movingPosition('next')">向后移动</p>
                      <p @click="movingPosition('delete')">删除</p>
                    </div>
                  </div>
                </div>
              </el-form-item>
            </el-col>
          </el-row>
        </el-form>
      </div>
      <!--评标专家-->
      <div class="page-title page-bg">
        02 评标专家
        <el-button v-if="!is_revokeLeader" size="mini" style="margin-left: 10px;" @click="setGroupLeader">设为组长</el-button>
        <el-button v-if="is_revokeLeader" size="mini" style="margin-left: 10px;" @click="revokeLeader">撤销组长</el-button>
        <el-button size="mini" style="margin-left: 10px;" @click="generateAccount">生成账号</el-button>
      </div>
      <div>
        <el-table
          ref="selectionTable"
          :data="tableData"
          stripe
          size="medium"
          class="el-table--striped-border"
          @selection-change="selectionClick">
          <el-table-column type="selection" width="55"/>
          <el-table-column label="序号" type="index" align="center" />
          <el-table-column label="姓名" prop="name" align="center" />
          <el-table-column label="身份证" prop="packName" align="center" />
          <el-table-column label="手机号" prop="name" align="center" />
          <el-table-column label="专家类型" prop="packName" align="center" />
          <el-table-column label="专家来源" prop="name" align="center" />
          <el-table-column label="是否设为组长" prop="packName" align="center" />
        </el-table>
      </div>
    </div>
    <!--专家分组-弹框-->
    <el-dialog
      :visible.sync="dialogExpertGrouping"
      title="专家分组"
      width="310px">
      <div>
        <el-form ref="form" label-width="68px">
          <el-row>
            <el-col :span="24">
              <el-form-item ref="linker" label="步骤名称" prop="linker">
                <el-input v-model="expertFormData.name" disabled placeholder="步骤名称"/>
              </el-form-item>
            </el-col>
            <el-col :span="24">
              <el-form-item ref="idCard" label="评标专家" prop="idCard">
                <template>
                  <el-checkbox-group v-model="expertFormData.checkList">
                    <el-checkbox label="00" style="margin-right: 10px;">所有专家</el-checkbox>
                    <el-checkbox label="01" style="margin-right: 10px;">商务专家</el-checkbox>
                    <el-checkbox label="02" style="margin-right: 10px;">技术专家</el-checkbox>
                    <el-checkbox label="03" style="margin-right: 10px;">经济专家</el-checkbox>
                  </el-checkbox-group>
                </template>
              </el-form-item>
            </el-col>
          </el-row>
        </el-form>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="expertSubmit">确 定</el-button>
        <el-button @click="expertCancel">取 消</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'BIdStepBack',
  data() {
    return {

      // 评标步骤部分
      dialogExpertGrouping: false, // 专家分组-弹框
      stepDatas: [// 评标步骤条
        {
          name: '签到',
          icon: 'iconfont icon-pingbiaoqiandao',
          checkList: ['00']
        }, {
          name: '特征码检查',
          icon: 'iconfont icon-tezhengmajiancha',
          checkList: ['00']
        }, {
          name: '限价评审',
          icon: 'iconfont icon-xianjiapingshen',
          checkList: ['00']
        }, {
          name: '资格审查',
          icon: 'iconfont icon-zigeshencha',
          checkList: ['00']
        }, {
          name: '否决投票',
          icon: 'iconfont icon-foujuetoupiao',
          checkList: ['00']
        }, {
          name: '符合性审查',
          icon: 'iconfont icon-fuhexiangshencha',
          checkList: ['00']
        }, {
          name: '商务评审',
          icon: 'iconfont icon-shangwupingshen',
          checkList: ['00']
        }, {
          name: '技术评审',
          icon: 'iconfont icon-jishupingshen',
          checkList: ['00']
        }, {
          name: '价格评审',
          icon: 'iconfont icon-jiagepingshen',
          checkList: ['00']
        }, {
          name: '推荐排名',
          icon: 'iconfont icon-tuijianpaiming',
          checkList: ['00']
        }, {
          name: '推荐中标候选人',
          icon: 'iconfont icon-tuijianzhongbiaohouxuanren',
          checkList: ['00']
        }, {
          name: '评审报表',
          icon: 'iconfont icon-pingshenbaobiao',
          checkList: ['00']
        }
      ],
      expertFormData: {}, // 专家分组
      editStep: {}, // 要编辑的步骤
      step: [], // 已设置的步骤
      eleStep: [], // 已设置的步骤元素
      downStep: false, // 选中正在拖动中的步骤
      menuData: {}, // 点击菜单传入的数据

      // 评标专家部分
      is_revokeLeader: false, // 是否为撤销组长状态
      is_startEvaluation: false, // 是否为评标开始状态
      selectionData: [], // 选择要设为组长的专家
      tableData: [// 评标专家列表
        {
          name: 1
        },
        {
          name: 2
        },
        {
          name: 3
        }
      ]
    }
  },
  mounted() {
  /* =======================评标步骤部分============================*/
    // 获取已设置的步骤元素
    this.eleStep = this.$refs.step.children
  },
  methods: {
    /* =======================评标步骤部分============================*/
    // 专家分组-点击要改变的步骤
    expertGroupingClick(item, index) {
      this.expertFormData = JSON.parse(JSON.stringify(item))
      this.expertFormData.index = index
      this.dialogExpertGrouping = true
    },
    // 专家分组确定
    expertSubmit() {
      this.dialogExpertGrouping = false
      this.stepDatas.splice(this.expertFormData.index, 1, this.expertFormData)
    },
    // 专家分组取消
    expertCancel() {
      this.dialogExpertGrouping = false
    },
    /* -------------------步骤编辑部分--------------------------*/
    // 添加步骤
    addStep(row, index) {
      this.stepDatas.splice(index, 1)
      this.step.push(row)
    },
    // 选择要编辑步骤
    editStepClick(item) {
      this.editStep = item
    },
    // 确定编辑步骤
    dEditStep() {
      // 效验
      if (!this.editStep.name) {
        this.$message({
          type: 'error',
          message: '此操作必须选择一个步骤进行编辑，请选择!'
        })
        return false
      }
      this.$prompt('步骤名称', '编辑名称', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        confirmButtonClass: 'confirmBtn',
        cancelButtonClass: 'cancelBtn',
        closeOnClickModal: false,
        inputValue: this.editStep.name// 步骤名称
      }).then(({ value }) => {
        this.$message({
          type: 'success',
          message: '操作成功!'
        })
        this.editStep.name = value
        // 清空对象
        this.editStep = {}
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '操作取消'
        })
        // 清空对象
        this.editStep = {}
      })
    },
    // 删除已选择的步骤
    deleteStep(row, index) {
      this.step.splice(index, 1)
      this.stepDatas.push(row)
      // 清空对象
      setTimeout(() => {
        this.editStep = {}
      })
    },
    // 点击选择要拖动的步骤移动位置
    mousedownStep(e, index, item) {
      // 移动样式
      const target = e.currentTarget
      const targetX = e.clientX - target.offsetLeft
      const targetY = e.clientY - target.offsetTop
      // 数据
      this.downStep = item
      this.$refs.step.onmousemove = (e) => {
        // 移动样式
        const left = e.clientX - targetX
        const top = e.clientY - targetY
        this.$refs.downStep.style.left = left + 'px'
        this.$refs.downStep.style.top = top + 'px'
        this.$refs.downStep.style.display = 'block'
        target.style.display = 'none'
      }
      document.onmouseup = (e) => {
        target.style.display = 'block'
        // 数据
        const step = JSON.parse(JSON.stringify(this.step))
        const stepData = step[index]// 当前选中的步骤
        const len = this.eleStep.length
        for (let i = 0; i < len; i++) {
          // 判断步骤位置移动情况
          if (this.$refs.downStep.offsetLeft < this.eleStep[i].offsetLeft + this.eleStep[i].offsetWidth / 2) {
            step.splice(index, 1)// 删除选中的步骤
            step.splice(i, 0, stepData)// 把删除的步骤重新复制过来
            this.step = step // 最终结果
            this.downStep = false
            // 样式清除
            this.$refs.downStep.style.display = 'none'
            this.$refs.step.onmousemove = null
            document.onmouseup = null
            return
          }
        }
        this.downStep = false
        // 样式清除
        this.$refs.downStep.style.display = 'none'
        this.$refs.step.onmousemove = null
        document.onmouseup = null
      }
    },
    /* -------------------点击右键弹出菜单部分----------------------*/
    // 点击右键弹出菜单
    menuClick(e, item, index) {
      this.menu = {
        item: item,
        index: index
      }
      this.$refs.menu.style.display = 'block'
      this.$refs.menu.style.left = e.clientX + 'px'
      this.$refs.menu.style.top = e.clientY + 'px'
      //  绑定一个滚动事件
      window.addEventListener('scroll', this.handleScroll, true)
    },
    // 滚动事件
    handleScroll() {
      this.$refs.menu.style.display = 'none'
      // 清空数据
      this.menu = {}
    },
    // 菜单操作事件
    movingPosition(ret) {
      if (ret === 'prev') { // 向前添加
        console.log(this.step[this.menu.index - 1])
        if (this.step[this.menu.index - 1]) {
          this.step.splice(this.menu.index, 1)
          this.step.splice(this.menu.index - 1, 0, this.menu.item)
        } else {
          this.$message({
            message: '此步骤已是最前置了，请选择其他步骤!'
          })
        }
      } else if (ret === 'next') { // 向后添加
        if (this.step[this.menu.index + 1]) {
          this.step.splice(this.menu.index, 1)
          this.step.splice(this.menu.index + 1, 0, this.menu.item)
        } else {
          this.$message({
            message: '此步骤已是最后了，请选择其他步骤!'
          })
        }
      } else if (ret === 'delete') { // 删除
        this.deleteStep(this.menu.item, this.menu.index)
      }
      this.$refs.menu.style.display = 'none'
      // 清空数据
      this.menu = {}
    },

    /* =======================评标专家部分============================*/

    // 选择要设为组长的专家
    selectionClick(val) {
      console.log(val)
      this.selectionData = val
    },
    // 设为组长 //最多只能设置一个专家
    setGroupLeader() {
      // 效验选择的专家
      if (this.selectionData.length > 1 || this.selectionData.length < 1) {
        this.$message({
          type: 'error',
          message: '此操作只能设置一个组长，请重新选择!'
        })
        this.selectionData = []
        return false
      }
      // 判断是否已评标显示对应的弹
      if (this.is_startEvaluation) {
        this.$confirm(`设定组长之后，该专家所有评标记录将全部清空，是否设定${this.selectionData[0].name}为组长？`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消'
        }).then(() => {
          this.$message({
            type: 'success',
            message: '操作成功!'
          })
          this.$refs.selectionTable.clearSelection()
        }).catch(() => {
        })
      } else if (!this.is_startEvaluation) {
        this.$confirm(`确认设定${this.selectionData[0].name}为组长吗?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消'
        }).then(() => {
          this.$message({
            type: 'success',
            message: '操作成功!'
          })
          this.is_revokeLeader = true
          this.is_startEvaluation = true
          this.$refs.selectionTable.clearSelection()
        }).catch(() => {
        })
      }
    },
    // 撤销组长
    revokeLeader() {
      // 判断是否已评标显示对应的弹
      if (this.is_startEvaluation) {
        this.$confirm(`撤销组长之后，该专家所有评标记录将全部清空，是否撤销${this.selectionData[0].name}的组长？`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消'
        }).then(() => {
          this.$message({
            type: 'success',
            message: '操作成功!'
          })
          this.$refs.selectionTable.clearSelection()
        }).catch(() => {
        })
      } else if (!this.is_startEvaluation) {
        this.$confirm(`确认撤销${this.selectionData[0].name}的组长吗?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消'
        }).then(() => {
          this.$message({
            type: 'success',
            message: '操作成功!'
          })
          this.$refs.selectionTable.clearSelection()
        }).catch(() => {
        })
      }
    },
    // 生成账号
    generateAccount() {
      this.$confirm('此操作将拒生成所有专家账号, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消'
      }).then(() => {
        this.$message({
          type: 'success',
          message: '操作成功!'
        })
      }).catch(() => {
      })
    }
  }
}
</script>

<style lang="less" scoped>
  @import "./../index.less";
  .evaluationSteps{
    padding: 0 32px;
    //评标步骤部分
    .step,.allSteps{
      ul{
        li{
          float: left;
          margin-right: 12px;
          cursor: pointer;
          .icon-add{
            margin-right: 4px;
            font-size: 12px;

          }
        }
      }
    }
    .step{
      position: relative;
      width: 100%;
      ul{
        position: relative;
        width: 100%;
        li:last-child{
          .ping_you{
            display: none;
          }
          .ping_x{
            top: -8px;
            right: -14px;
          }
        }
      }
      .downStep,li{
        -moz-user-select: none;
        -webkit-user-select: none;
        -ms-user-select: none;
        -khtml-user-select: none;
        user-select: none;
        margin-bottom: 4px;
        position: relative;
        cursor: pointer;
        i{
          display: inline-block;
          width: 30px;
          height: 30px;
          border-radius: 50%;
          font-size: 20px;
          color: #fff;
          background-color: @gray;
          line-height: 30px;
          text-align: center;
          margin-right: 4px;
        }
        .ping_you{
          display: inline-block;
          height: 18px;
          width: 18px;
          line-height: 18px;
          border-radius: 50%;
          color: @gray;
          text-align: center;
          margin-left: 10px;
        }
        .ping_x{
          position: absolute;
          top: -8px;
          right: 18px;
          font-size: 6px;
          width: 12px;
          height: 12px;
          color: @gray;
          margin-left: 4px;
          border-radius: 50%;
        }
      }
      .downStep{
        display: none;
        position: absolute;
      }
      .menu{
        z-index: 1000;
        border-radius:1px;
        position: fixed;
        width: 110px;
        padding: 6px 0;
        box-sizing: border-box;
        height: auto;
        background: #fff;
        border: 1px solid @french-grey;
        box-shadow: 0px 0px 1px @gray;
        text-align: center;
        display: none;
        p:hover{
          background: @french-grey;
          cursor: pointer;
        }
      }
    }
  }
</style>
