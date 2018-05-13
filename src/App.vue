<template>
	<div id="app">
		<img src="./assets/logo.png">
		<HelloWorld/>
	</div>
</template>

<script>
import HelloWorld from './components/HelloWorld'
import NebPay from 'nebPay'
export default {
    name: 'App',
    components: {
        HelloWorld
    },
    created() {
        const to = 'n1wfNuDbm956s87bG2KHcENo84Ez2k412CQ'
        const value = '0'
        const func = 'verifyAddress'
        const defaultOptions = {
            goods: {
                //Dapp端对当前交易商品的描述信息，app暂时不展示
                name: '', //商品名称
                desc: '', //描述信息
                orderId: '', //订单ID
                ext: '' //扩展字段
            },
            qrcode: {
                showQRCode: false, //是否显示二维码信息
                container: undefined //指定显示二维码的canvas容器，不指定则生成一个默认canvas
            },
            // callback 是记录交易返回信息的交易查询服务器地址，不指定则使用默认地址
            callback: undefined,
            // listener: 指定一个listener函数来处理交易返回信息（仅用于浏览器插件，App钱包不支持listener）
            listener: undefined,
            // if use nrc20pay ,should input nrc20 params like name, address, symbol, decimals
            nrc20: undefined
        }
        var options = {
            callback: undefined //可以指定交易查询服务器
        }

        // 拼凑addCommodity的参数格式
        const params = {
            name: 'name',
            seller: 'seller',
            seller_email: 'zaakin',
            receipt_address: 'piouslove',
            price: 4,
            total_amount: 10,
            remained_amount: '1133664',
            description: 'only for test',
            link: 'love'
        }
        const {name,seller,seller_email,receipt_address,price,total_amount,remained_amount,description,link} = params
        const addCommodityArgs = "[\"" + name + "\",\"" + seller + "\",\"" + seller_email + "\",\"" + receipt_address + "\",\"" + price + "\",\"" + total_amount + "\",\"" + remained_amount + "\",\"" + description + "\",\"" + link + "\"]"

        // 拼凑 verifyAddress的参数格式
        const verifyAddressArgs = "[\"" + to + "\"]"

        //实例化
        const neb = new NebPay()
        // 调用call  返回交易流水号
        const serialNumber = neb.call(to, value, func,verifyAddressArgs, defaultOptions)
        //打印返回值：serialNumber
        console.log(serialNumber)

        // 调用pay
        // neb.pay(to, value, defaultOptions)

    }
}
</script>

<style>
#app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
}
</style>

