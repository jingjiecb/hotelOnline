<template>
    <a-modal
        :visible="orderModalVisible"
        title="预定详情"
        cancelText="取消"
        okText="下单"
        @cancel="cancelOrder"
        @ok="handleSubmit"
    >
        <a-form :form="form">
            <a-form-item v-bind="formItemLayout" label="房型信息">
                <span>{{ currentOrderRoom.roomType }}</span>
            </a-form-item>
            <a-form-item v-bind="formItemLayout" label="入住日期">
                <a-range-picker
                    format="YYYY-MM-DD"
                    @change="changeDate"
                    v-decorator="[
                        'date', 
                        {
                            rules: [{ required: true, message: '请选择入住时间' }]   
                        }
                    ]"
                    :placeholder="['入住日期','退房日期']"
                />
            </a-form-item>
            <a-form-item v-bind="formItemLayout" label="入住人数">
                <a-select
                    v-decorator="[
                        'peopleNum',
                        { rules: [{ required: true, message: '请选择入住人数' }] },
                    ]"
                    placeholder="请选择入住人数"
                    @change="changePeopleNum"
                >
                    <a-select-option :value="1">
                    1
                    </a-select-option>
                    <a-select-option :value="2">
                    2
                    </a-select-option>
                     <a-select-option :value="3">
                    3
                    </a-select-option>
                    <a-select-option :value="4">
                    4
                    </a-select-option>
                </a-select>
            </a-form-item>
            <a-form-item v-bind="formItemLayout" label="有无儿童">
                <a-radio-group
                    v-decorator="[
                        'haveChild',
                        { rules: [{required: true, message: '请选择有无儿童入住', }] }
                    ]"
                >
                    <a-radio :value="1">有</a-radio>
                    <a-radio :value="0">无</a-radio>
                </a-radio-group>
            </a-form-item>
            <a-form-item v-bind="formItemLayout" label="房间数">
                <a-input-number id="inputNumber" :min="1" :max="10" @change="changeRoomNum"
                                v-decorator="[
                        'roomNum',
                        { rules: [{ required: true, message: '请输入房间数' }] },
                    ]"
                />
            </a-form-item>
            <a-form-item v-bind="formItemLayout" label="房间单价">
                <span>{{ currentOrderRoom.price }}</span>
            </a-form-item>
            <a-form-item v-bind="formItemLayout" label="原总价">
                <span>￥{{ sumPrice | numFilter }}</span>
            </a-form-item>
            <a-form-item v-bind="formItemLayout" label="会员折后价" v-if="userInfo.vipType!=null">
                <span>￥{{ totalPrice | numFilter }}</span>
            </a-form-item>
            <a-divider></a-divider>
            <h2 v-if="orderMatchCouponList.length>0">优惠</h2>
            <a-checkbox-group v-model="checkedList" @change="onchange">
                <a-table
                    :columns="columns"
                    :dataSource="orderMatchCouponList"
                    :showHeader="false"
                    bordered
                    v-if="orderMatchCouponList.length>0"
                >
                    <a-checkbox
                        slot="id"
                        slot-scope="record"
                        :value="record"
                    >
                    </a-checkbox>
                </a-table>
            </a-checkbox-group>
             <a-form-item v-bind="formItemLayout" label="券后最终价">
                <span>￥{{ finalPrice | numFilter}}</span>
            </a-form-item>
        </a-form>
    </a-modal>
</template>
<script>
import { mapGetters, mapMutations, mapActions } from 'vuex'
import {message} from "ant-design-vue";
const moment = require('moment')
const columns = [
    {  
        title: '勾选',
        dataIndex: 'id',
        scopedSlots: {customRender: 'id'}
    },
    {
        title: '优惠类型',
        dataIndex: 'couponName',
        scopedSlots: {customRender: 'couponName'}
    },
    {
        title: '折扣',
        dataIndex: 'discount',
    },

    {
        title: '优惠简介',
        dataIndex: 'description',
    },
    {
        title: '优惠金额',
        dataIndex: 'discountMoney',
    },
  ];
export default {
    name: 'orderModal',
    data() {
        return {
            formItemLayout: {
                labelCol: {
                    xs: { span: 12 },
                    sm: { span: 6 },
                },
                wrapperCol: {
                    xs: { span: 24 },
                    sm: { span: 16 },
                },
            },
            sumPrice:'',
            totalPrice: '',
            columns,
            checkedList: [],
            finalPrice: '',
            roomNumber: 1
        }
    },
    computed: {
        ...mapGetters([
            'orderModalVisible',
            'currentOrderRoom',
            'currentHotelId',
            'currentHotelInfo',
            'userId',
            'userInfo',
            'orderMatchCouponList'
        ]),
        
    },
    beforeCreate() {
        this.form = this.$form.createForm(this, { name: 'orderModal' });
    },
    methods: {
        ...mapMutations([
            'set_orderModalVisible'
        ]),
        ...mapActions([
            'addOrder',
            'getOrderMatchCoupons'
        ]),
        cancelOrder() {
            this.set_orderModalVisible(false)
        },
        confirmOrder() {

        },
        changeDate(v) {
            if(this.totalPrice != ''){
                this.totalPrice = this.form.getFieldValue('roomNum') * moment(v[1]).diff(moment(v[0]), 'day') * Number(this.currentOrderRoom.price)
            }
        },
        changePeopleNum(v){

        },
        changeRoomNum(v) {
            console.log(this.userInfo)
            if(moment(this.form.getFieldValue('date')[1]).diff(moment(this.form.getFieldValue('date')[0]),'day')===0){
                this.totalPrice = Number(v) * Number(this.currentOrderRoom.price)
            }
            else{
                this.totalPrice = Number(v) * Number(this.currentOrderRoom.price) * moment(this.form.getFieldValue('date')[1]).diff(moment(this.form.getFieldValue('date')[0]),'day')
            }
            this.sumPrice=this.totalPrice
            if(this.userInfo.vipType=='普通会员'){
                this.totalPrice=this.totalPrice*0.9
            }
            else if(this.userInfo.vipType=='企业会员'){
                this.totalPrice=this.totalPrice*0.8
            }
            this.finalPrice = this.totalPrice
        },
        onchange() {
            this.finalPrice = this.totalPrice

            if(this.checkedList.length>0){
                this.orderMatchCouponList.filter(item => this.checkedList.indexOf(item.id)!=-1).forEach(item => this.finalPrice= this.finalPrice-item.discountMoney)
            }else{
                
            }
            if(this.finalPrice<0){
                this.finalPrice=0
            }
        },
        handleSubmit(e) {
            this.roomNumber=1;
            e.preventDefault();
            this.form.validateFieldsAndScroll((err, values) => {
                if (!err) {
                    if(moment(this.form.getFieldValue('date')[0]).isAfter(moment().startOf('day'))) {
                        const data = {
                            hotelId: this.currentHotelId,
                            hotelName: this.currentHotelInfo.name,
                            userId: Number(this.userId),
                            checkInDate: moment(this.form.getFieldValue('date')[0]).format('YYYY-MM-DD'),
                            checkOutDate: moment(this.form.getFieldValue('date')[1]).format('YYYY-MM-DD'),
                            roomType: this.currentOrderRoom.roomType == '大床房' ? 'BigBed' : this.currentOrderRoom.roomType == '双床房' ? 'DoubleBed' : 'Family',
                            roomNum: this.form.getFieldValue('roomNum'),
                            peopleNum: this.form.getFieldValue('peopleNum'),
                            haveChild: this.form.getFieldValue('haveChild'),
                            createDate: '',
                            price: this.checkedList.length > 0 ? this.finalPrice : this.totalPrice
                        }
                        this.addOrder(data)
                    }
                    else{
                        message.error('预订日期不合法，预订失败')
                    }
                }
            });
        },
        numPlus() {
            this.roomNumber++
        },
        numSub(){
            if (this.roomNumber>0) this.roomNumber--
        }
    },
    watch:{
        totalPrice(val) {
            let data = {
                userId: this.userId,
                hotelId: this.currentHotelId,
                orderPrice: this.totalPrice,
                roomNum: this.form.getFieldValue('roomNum'),
                checkIn: moment(this.form.getFieldValue('date')[0]).format('YYYY-MM-DD'),
                checkOut: moment(this.form.getFieldValue('date')[1]).format('YYYY-MM-DD'),
            }
            this.getOrderMatchCoupons(data)
        }
    },
    filters: {
        numFilter (value) {
            let realVal = ''
            if (!isNaN(value) && value!== '') {
                // 截取当前数据到小数点后两位
                realVal = parseFloat(value).toFixed(2)
            } else {
                realVal = '--'
            }
            return realVal
        }
    }
}
</script>