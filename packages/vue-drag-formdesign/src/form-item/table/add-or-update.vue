<template>
  <el-dialog
    :title="!dataForm._id ? '新增' : '修改'"
    :close-on-click-modal="false"
     :append-to-body="true" 
    :lock-scroll="false"
    :visible.sync="visible"> 
   <!--  <div class="mod-config"> -->
      <!-- 
   <FormBuild  :formTemplate="formTemplate" :renderPreview="renderPreview" :models="dataForm" /> -->
   <el-form
      v-if="
        typeof formTemplate.list !== 'undefined' && typeof formTemplate.config !== 'undefined'
      "
      class="form-build form-design"
      :label-position="formTemplate.config.labelPosition"
      :hide-required-asterisk="formTemplate.config.hideRequiredMark" 
      :label-width="formTemplate.config.labelWidth + 'px'" 
      ref="dataForm" 
      :model="dataForm"  
      size="mini"
    > 
    <template
      v-for="(item,index) in formTemplate.list"  
    >
      <el-form-item v-if="
      !(item.options.hidden === true) &&
        [
          'input',
          'textarea',
          'date',
          'time',
          'number',
          'radio',
          'checkbox',
          'select',
          'rate',
          'switch',
          'slider',
          'uploadImg',
          'uploadFile',
          'cascader'
        ].includes(item.type) && dynamicVisibleItem(item)  "
        :key="index"
        :label="formTemplate.config.labelWidth > 0 ? item.label : null " 
        :rules="recordRules(item)"
        :prop="item.rules && item.rules.length > 0 ? item.model : null"
      >   


       <el-tooltip 
        class="item" 
        effect="light" 
        :enterable="false"
        :open-delay="500"
        v-if="item.options.tooptip && item.options.tooptip.trim()" 
        placement="top-start">
          <div slot="content" class="tooltip-content"> {{item.options.tooptip}}</div>
          <BaseItem 
            :models="dataForm"  
            :formConfig="formTemplate.config"
            :renderPreview="renderPreview"
            
            :record="item"
            :disabled="disabled || item.options.disabled"

            /> 
        </el-tooltip>  
        <BaseItem 
          v-else
            :models="dataForm"  
            :formConfig="formTemplate.config"
            :renderPreview="renderPreview"
            
            :record="item"
            :disabled="disabled || item.options.disabled"

            /> 
      </el-form-item>

      
    </template>
     <el-form-item label="排序" prop="seq">
          <template v-if="renderPreview">
            {{dataForm.seq}}
          </template>
          <template v-else>
            <el-input-number v-model="dataForm.seq" controls-position="right" :min="0" label="排序号" :disabled="renderPreview"></el-input-number>
          </template>
         
      </el-form-item>
      
</el-form>
   

   <!--   <form-item
        v-for="(item,index) in formTemplate.list" 
          :disabled="item.options.disabled" 
          :renderPreview="renderPreview"
          :models.sync="dataForm"  
          :key="item.key"
          :record="item"
          :formConfig="formTemplate.formConfig" 
        />

    </el-form> -->


    <div  class="mod-footer">
      <el-button @click="visible = false">取消</el-button>
     <el-button :disabled="loading" v-if="!renderPreview" type="primary" @click="dataFormSubmit()">确定</el-button>
    </div>
 <!--  </div> -->
  </el-dialog> 
</template>

<script>
//import FormBuild from '../../form-build/index' 
  import cloneDeep from 'lodash/cloneDeep'
  import BaseItem from '../BaseItem'
  import {dynamicFun} from '../../utils'
  export default {
    name: 'table-add-or-update' ,
    components: {
      //FormBuild
      BaseItem
    },  
    data () {
      return {
        loading: false,
        visible: false,
        dataForm: {
          _id: '',
          seq: 0
        }, 
        dataRule: { 
        }
      }
    },
    props: {
      formTemplate: {
        type: Object, 
        default: () => ({})
      },
         // 是否预览结果表单
      renderPreview: {
        type: Boolean ,
        default: false
      },
      disabled: {
        type: Boolean,
        default: false
      }
    },  
    methods: {
      recordRules(record){
        // 2020-07-29 如果是预览 不需要规则验证
        if(this.renderPreview) {
          return []
        }
        const rules = record.rules  

        // 循环判断
        for(var i = 0 ; i < rules.length ; i++){
          const t = rules[i]
           if(t.vtype == 1 || t.vtype == 2){ 
            t.validator =  this.validatorFiled 
          } 

          // 判断trigger
          if(!t.trigger) {
            t.trigger =  ['change','blur']
          }
        }
       

        return rules 

      },
      dynamicVisibleItem(record){ 
      
        if(!record.options || !record.options.dynamicVisible){
          return true
        }
        if(!record.options.dynamicVisibleValue){
          return true
        }
        let fstr = record.options.dynamicVisibleValue;
        // 打开了开关 这里获取函数内容
        const func =  dynamicFun(fstr , this.dataForm)
        return func
      },
       validatorFiled (rule , value , callback) {
      
        // 判断rule类型 
        if(rule.vtype == 1) {
          // 正则
          if(!rule.pattern) {
            callback()
            return
          }
          // 正则匹配
          var patt1=new RegExp(rule.pattern);
          //document.write(patt1.test("free"));

          if(patt1.test(value)) {
            callback() 
           } else {
            callback(new Error(rule.message)) 
           }

           return
        } else if(rule.vtype == 2) {
          // 表达式
          const script = rule.script

          // 打开了开关 这里获取函数内容
         const fvalue =  dynamicFun(script , this.dataForm)
          
          if (!fvalue) {
            callback(new Error(rule.message))
          } else {
            callback()
          }


        }

       
      } ,
       
      init (data) {
          this.visible = true 
          this.dataForm._id = null  
          if(data) {
            //this.dataForm = data
            for(var i in data){
              this.dataForm[i] = data[i]
            }
          } else {
            // 初始化数据  
             
            const d = {}
            this.formTemplate.list.forEach(item => {
              if(item.options.defaultValue)
                d[item.model] = item.options.defaultValue;
              else 
                d[item.model] = null
            }); 
 
            this.dataForm = {_id:null,seq: 0 , ...d}

            this.$nextTick(() => {
              this.$refs['dataForm'].resetFields()
         
            })
          }
             

        },
        // 表单提交
        dataFormSubmit () {
          this.$refs['dataForm'].validate((valid) => {
            if (valid) { 
              this.loading = true
              if(!this.dataForm._id) {
                // 回填一个ID 
                const id = new Date().getTime() * 10 + parseInt(Math.random() * 100)
                this.dataForm._id = id 
                this.$emit('formAdd' , cloneDeep(this.dataForm))
              } else {
                this.$emit('formUpdate' , cloneDeep(this.dataForm))
              }
              this.loading = false
              this.visible = false
            }
          })

 
        }
    }
  }
</script>
