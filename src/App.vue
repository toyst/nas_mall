<template>
	<div id="app">
                    <v-btn
              color="pink"
              dark
              small
              fixed
              bottom
              left
              fab
              style="left: 30vw"
              @click.native="dialog=true"
            >
              +
            </v-btn>
		<v-tabs icons-and-text
		        centered
		        dark
		        color="cyan">
			<v-tabs-slider color="yellow"></v-tabs-slider>
			<v-tab href="#tab-1">
				所有商品
				<!-- <v-icon>view_list</v-icon> -->
			</v-tab>
			<v-tab href="#tab-2">
				购买记录
				<!-- <v-icon>history</v-icon> -->
			</v-tab>
			<v-tab-item id="tab-1">
				<div class="goods-list">
					<v-card v-for="(item,index)  in items"
					        width="48%"
					        :key="index"
					        style="margin-bottom: 20px">
						<v-card-media class="white--text"
						              height="230px"
						              :src="item.src || '/static/plane.jpg'">
							<v-container fill-height
							             fluid>
								<v-layout fill-height>
									<v-flex xs12
									        align-end
									        flexbox>
										<span class="headline">{{item.name}}</span>
									</v-flex>
								</v-layout>
							</v-container>
						</v-card-media>
						<v-card-title>
							<div>
								<span class="grey--text">{{item.name}}</span><br>
								<span>{{item.desc}}</span><br>
								<span>￥{{item.price}}</span><br>
							</div>
						</v-card-title>
						<v-card-actions>
							<v-btn flat
							       color="orange">购买</v-btn>
						</v-card-actions>
					</v-card>
				</div>
			</v-tab-item>
			<v-tab-item id="tab-2">

			</v-tab-item>

		</v-tabs>
        <v-dialog v-model="dialog" max-width="500px" >
            <v-card style="background: white">
					<v-card-title>
						<span class="headline">添加商品</span>
					</v-card-title>
					<v-card-text>
						<v-container grid-list-md>
							<v-layout wrap>
								<v-flex xs12
								        sm6
								        md4>
									<v-text-field label="商品名称" v-model="name"
									              required></v-text-field>
								</v-flex>
								<v-flex xs12
								        sm6
								        md4>
									<v-text-field label="价格（NAS）" v-model="price"
									              hint="请输入整数"></v-text-field>
								</v-flex>
								<v-flex xs12>
									<v-text-field label="图片地址" v-model="link"
									              required></v-text-field>
								</v-flex>
								<v-flex xs12>
									<v-text-field label="描述" v-model="desc"
									              required></v-text-field>
								</v-flex>
								<v-flex xs12>
									<v-text-field label="Email" v-model="email"
									              required></v-text-field>
								</v-flex>
							</v-layout>
						</v-container>
					</v-card-text>
					<v-card-actions>
						<v-spacer></v-spacer>
						<v-btn color="blue darken-1"
						       flat
						       @click.native="dialog = false">取消</v-btn>
						<v-btn color="blue darken-1"
						       flat
						       @click.native="handleOk">确定</v-btn>
					</v-card-actions>
				</v-card>
        </v-dialog>

	</div>
</template>

<script>
import NebPay from 'nebPay'
import nebulas from 'nebulas'
const nebPay = new NebPay()
const Account = nebulas.Account
const neb = new nebulas.Neb()
const from = Account.NewAccount().getAddressString()
const to = 'n1oYYjxpUf1UD7Fg3v5vjydgV9QjWKqBSD3'
neb.setRequest(new nebulas.HttpRequest('https://testnet.nebulas.io'))

export default {
    name: 'App',
    data() {
        return {
            dialog: false,
            name: '',
            price: '',
            desc: '',
            email: '',
            link: '',
            items: [
                { name: '商品', desc: '描述', price: 20 },
                { name: '商品', desc: '描述', price: 20 },
                { name: '商品', desc: '描述', price: 20 },
                { name: '商品', desc: '描述', price: 20 },
                { name: '商品', desc: '描述', price: 20 }
            ]
        }
    },
    methods: {
        handleOk() {
            // nebPay.call()
        }
    }
    // created() {
    //     const to = 'n1wfNuDbm956s87bG2KHcENo84Ez2k412CQ'
    //     const value = '0'
    //     const func = 'verifyAddress'
    //     const defaultOptions = {
    //         goods: {
    //             //Dapp端对当前交易商品的描述信息，app暂时不展示
    //             name: '', //商品名称
    //             desc: '', //描述信息
    //             orderId: '', //订单ID
    //             ext: '' //扩展字段
    //         },
    //         qrcode: {
    //             showQRCode: false, //是否显示二维码信息
    //             container: undefined //指定显示二维码的canvas容器，不指定则生成一个默认canvas
    //         },
    //         // callback 是记录交易返回信息的交易查询服务器地址，不指定则使用默认地址
    //         callback: undefined,
    //         // listener: 指定一个listener函数来处理交易返回信息（仅用于浏览器插件，App钱包不支持listener）
    //         listener: undefined,
    //         // if use nrc20pay ,should input nrc20 params like name, address, symbol, decimals
    //         nrc20: undefined
    //     }
    //     var options = {
    //         listener: function (res) {
    //             console.log('交易返回信息',res)
    //         },
    //         callback: undefined //可以指定交易查询服务器
    //     }

    //     // 拼凑addCommodity的参数格式
    //     const params = {
    //         name: 'name',
    //         seller: 'seller',
    //         seller_email: 'zaakin',
    //         receipt_address: 'piouslove',
    //         price: 4,
    //         total_amount: 10,
    //         remained_amount: '1133664',
    //         description: 'only for test',
    //         link: 'love'
    //     }
    //     const {name,seller,seller_email,receipt_address,price,total_amount,remained_amount,description,link} = params
    //     const addCommodityArgs = "[\"" + name + "\",\"" + seller + "\",\"" + seller_email + "\",\"" + receipt_address + "\",\"" + price + "\",\"" + total_amount + "\",\"" + remained_amount + "\",\"" + description + "\",\"" + link + "\"]"

    //     // 拼凑 verifyAddress的参数格式
    //     const verifyAddressArgs = "[\"" + to + "\"]"

    //     //实例化
    //     const neb = new NebPay()
    //     // 调用call  返回交易流水号
    //     const serialNumber = neb.call(to, value, func,verifyAddressArgs, options)
    //     //打印返回值：serialNumber
    //     console.log(serialNumber)
    //            //设置定时查询交易结果
    //     var intervalQuery = setInterval(function() {
    //         funcIntervalQuery();
    //     }, 5000);
    //     // 调用pay
    //     // neb.pay(to, value, defaultOptions)

    //     window.addEventListener('message',function (e) {
    //         console.log(e.data,'data!!')
    //     })

    //     function funcIntervalQuery() {
    //         neb.queryPayInfo(serialNumber)   //search transaction result from server (result upload to server by app)
    //             .then(function (resp) {
    //                 var respObject = JSON.parse(resp)
    //                 console.log("交易结果 result: " , respObject)   //resp is a JSON string
    //                 if(respObject.code === 0){
    //                     //交易成功, 处理相关任务

    //                     clearInterval(intervalQuery)    //清除定时查询
    //                 }
    //             })
    //             .catch(function (err) {
    //                 console.log(err);
    //             });
    //     }
    // }
}
</script>

<style>
#app {
    min-height: 100vh;
    width: 80vh;
    margin: 0 auto;
    background: white;
}
.goods-list {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    padding: 20px;
    height: 100%;
    /* overflow: auto; */
}
</style>

