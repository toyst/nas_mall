<template>
	<div id="app">
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
				添加商品
				<!-- <v-icon>history</v-icon> -->
			</v-tab>
            <v-tab href="#tab-3">
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
								<span>{{item.description}}</span><br>
								<span>￥{{item.price}}</span><br>
							</div>
						</v-card-title>
						<v-card-actions>
							<v-btn flat
							       color="orange" @click.native="buy(item.id)">购买</v-btn>
                           	<v-text-field label="数量" v-model="buyCount"
									              hint="请输入整数"></v-text-field>
						</v-card-actions>
					</v-card>
				</div>
			</v-tab-item>
			<v-tab-item id="tab-2">
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
                                <v-flex xs12
								        sm6
								        md4>
									<v-text-field label="数量）" v-model="count"
									              hint="请输入整数"></v-text-field>
								</v-flex>
								<v-flex xs12>
									<v-text-field label="图片地址" v-model="link"
									              required></v-text-field>
								</v-flex>
								<v-flex xs12>
									<v-text-field label="描述" v-model="description"
									              required></v-text-field>
								</v-flex>
								<v-flex xs12>
									<v-text-field label="seller_email" v-model="seller_email"
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
			</v-tab-item>
            <v-tab-item id="tab-3">
                购买记录
            </v-tab-item>
		</v-tabs>


	</div>
</template>

<script>
import NebPay from "nebPay";
import nebulas from "nebulas";
const nebPay = new NebPay();
const Account = nebulas.Account;
const neb = new nebulas.Neb();
const from = Account.NewAccount().getAddressString();
const to = "n22T3E3aDorqs9aC92hTAQcLDpyyM93xds4";
neb.setRequest(new nebulas.HttpRequest("https://testnet.nebulas.io"));

const getParams = arr => {
  console.log(arr, "arr");
  let str = '["';
  arr.forEach((v, idx) => {
    if (idx < arr.length - 1) {
      str += v + '","';
    } else {
      str += v + '"]';
    }
  });
  return str;
};
const call = ({ func, callArgs }) => {
  var value = "0";
  var nonce = "0";
  var gas_price = "1000000";
  var gas_limit = "2000000";
  var callFunction = func;
  var contract = {
    function: callFunction,
    args: callArgs
  };

  return neb.api.call(from, to, value, nonce, gas_price, gas_limit, contract);
};

export default {
  name: "App",
  data() {
    return {
      buyCount: "",
      dialog: false,
      name: "",
      price: "",
      description: "",
      seller_email: "",
      link: "",
      count: "",
      items: []
    };
  },
  methods: {
    handleOk() {
      const { name, description, price, seller_email, count, link } = this;
      const func = "addCommodity";
      const value = "0";
      const addCommodityArgs = getParams([
        name,
        seller_email,
        price,
        count,
        description,
        link
      ]);
      const options = {
        listener: function(res) {
          console.log("交易返回信息", res);
        },
        callback: undefined //可以指定交易查询服务器
      };
      nebPay.call(to, value, func, addCommodityArgs, options);
      this.name = this.description = this.price = this.seller_email = this.count = this.link =
        "";
    },
    getItems() {
      call({
        func: "getCommodities",
        callArgs: getParams([""])
      }).then(res => {
        this.items = JSON.parse(res.result);
      });
    },
    getRecord() {
        call({
        func: "getTxes",
        callArgs: getParams([""])
      }).then(res => {
        console.log('记录',JSON.parse(res.result))
      });
    },
    buy(key) {
      const { buyCount } = this;
      const func = "buy";
      const value = "0";
      const args = getParams([key,buyCount]);
      const options = {
        listener: function(res) {
          console.log("交易返回信息", res);
        },
        callback: undefined //可以指定交易查询服务器
      };
      nebPay.call(to, value, func, args, options);
    }
  },
  created() {
    this.getItems();
    this.getRecord()
  }
  // created() {
  //     const to = 'n1wfNuDbm956s87bG2KHcENo84Ez2k412CQ'
  //     const value = '0'
  //     const func = 'verifyAddress'
  //     const defaultOptions = {
  //         goods: {
  //             //Dapp端对当前交易商品的描述信息，app暂时不展示
  //             name: '', //商品名称
  //             description: '', //描述信息
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
  //         seller_seller_email: 'zaakin',
  //         receipt_address: 'piouslove',
  //         price: 4,
  //         total_amount: 10,
  //         remained_amount: '1133664',
  //         descriptionription: 'only for test',
  //         link: 'love'
  //     }
  //     const {name,seller,seller_seller_email,receipt_address,price,total_amount,remained_amount,descriptionription,link} = params
  //     const addCommodityArgs = "[\"" + name + "\",\"" + seller + "\",\"" + seller_seller_email + "\",\"" + receipt_address + "\",\"" + price + "\",\"" + total_amount + "\",\"" + remained_amount + "\",\"" + descriptionription + "\",\"" + link + "\"]"

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
};
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
