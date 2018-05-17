<template>
	<div id="app">
        <mu-tabs :value="activeTab" @change="handleTabChange">
            <mu-tab value="tab1" icon="list" title="商品列表"/>
            <mu-tab value="tab2" icon="history" title="购买记录"/>
        </mu-tabs>
        <div v-if="activeTab === 'tab1'" class="goods-list">
            <div v-for="item in items" :key="item.id">
                <mu-card>
                    <mu-card-header :title="item.name">
                    </mu-card-header>
                    <mu-card-media :subTitle="item.description">
                        <img :src="item.link || '/static/plane.jpg'" />
                    </mu-card-media>
                    <mu-card-text>
                        <span>￥{{item.price}}</span> <br>
                        <span>库存: {{item.remained_quantity}} / {{item.total_quantity}}</span><br>
                        <span>卖家邮箱： {{item.seller_email}}</span>
                    </mu-card-text>
                    <mu-card-actions>
                        <mu-raised-button label="购买"  secondary @click.native="dialog = true;currItem = item;" />
                    </mu-card-actions>
                </mu-card>
            </div>
        </div>
        <div v-if="activeTab === 'tab2'" class="record">
            <ul>
                <li v-for="(item,idx) in record" :key="idx">
                    <div class="record-icon">
                        <mu-icon value="card_giftcard" color="blue" :size="48" />
                    </div>
                    <div class="record-main">

                        <span class="record-name">{{item.name}} <br></span>
                        <span class="record-price">￥ {{item.price}} <span class="record-unit">(NAS)</span> <br></span>
                        
                    </div>
                    <div class="record-sub">
                        <span>数量： {{item.amount}} <span class="record-unit">(NAS)</span> </span> <br>
                        <span>总价：{{item.quantity}} <span class="record-unit">(NAS)</span> <br></span>
                        <span>时间： {{new Date(item.timestamp * 1000).toLocaleString()}}</span>
                    </div>
                </li>
            </ul>
        </div>
        <mu-dialog :open="dialog" title="输入购买数量" @close="handleCloseBuy">
            <mu-text-field hintText="提示文字" v-model="buyCount" />
            <mu-flat-button slot="actions" @click="handleCloseBuy" primary label="取消"/>
            <mu-flat-button slot="actions" primary @click="handleBuy" label="确定"/>
        </mu-dialog>
        <mu-dialog :open="addDialog" title="添加商品" @close="handleCloseAdd">
            <mu-text-field label="名称" style="width: 30%" labelFloat v-model="name"/> 
            <mu-text-field label="价格" style="width: 30%" labelFloat v-model="price"/>
            <mu-text-field label="数量" style="width: 30%" labelFloat v-model="count"/>
            <br/>
            <mu-text-field  multiLine style="width: 90%" :rows="3" :rowsMax="6" label="描述"  labelFloat v-model="description" /> <br>
            <!-- <br/> -->
            <mu-text-field label="邮箱" labelFloat style="width: 30%" v-model="seller_email"/>
            <mu-text-field label="图片链接" labelFloat style="width: 30%" v-model="link"/><br/>
            <mu-flat-button slot="actions" @click="handleCloseAdd" primary label="取消"/>
            <mu-flat-button slot="actions" primary @click="handleAdd(item.id,item.price)" label="确定"/>
        </mu-dialog>
        <mu-float-button icon="add" primary  style="position: fixed;bottom: 35px; right: 31vw" @click="addDialog = true" />
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
        currItem: {},
        addDialog: false,
      activeTab: "tab1",
      buyCount: 0,
      dialog: false,
      name: "",
      price: "",
      description: "",
      seller_email: "",
      link: "",
      count: "",
      items: [],
      record: []
    };
  },
  watch: {
      activeTab(value) {
          if(value === 'tab1') {
              this.getItems()
          }
          if(value === 'tab2') {
              this.getRecord()
          }
      }
  },
  methods: {
    handleAdd() {
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
        console.log("记录", JSON.parse(res.result));
        const arr = JSON.parse(res.result)
        const record = []
        arr.forEach(v => {
            if(v.buyer === Account.NewAccount().getAddressString()) {
                record.push(v)
            }
        })
        console.log(arr,'记录')
        this.record = arr
      });
    },
    handleBuy() {
      const { buyCount,currItem: {id,price} } = this;
      const func = "buy";
      const value = String((price * buyCount) + 0.01)
      const args = getParams([id, buyCount]);
      const options = {
        listener: function(res) {
          console.log("交易返回信息", res);
        },
        callback: undefined //可以指定交易查询服务器
      };
      nebPay.call(to, value, func, args, options)
      this.handleCloseBuy()
    },
    handleCloseBuy() {
        this.dialog = false
        this.buyCount = 0
        this.currItem = {}
    },
    handleTabChange(val) {
      this.activeTab = val;
    },
    handleCloseAdd() {
        this.addDialog = false
    },
    handleAdd() {
        this.addDialog = false
    }
  },
  created() {
    this.getItems();
    // this.getRecord();
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

<style scoped lang="scss">
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
  > div {
      width: 48%;
      margin-bottom: 20px;
  }
}
.record {
    width: 100%;
    padding: 20px;
    >ul {
        width: 100%;
        >li {
            list-style: none;
            width: 100%;
            padding: 0 20px;
            height: 100px;
            box-shadow: 0 0 24px 0 rgba(15, 66, 76, 0.25);
            display: flex;
            justify-content: space-between;
            align-items: center;
            .record-unit {
                color: #bbb;
                margin-left: 5px;
                font-size: 12px;
            }
            .record-icon {
                width: 10%;
            }
            .record-main {
                width: 60%;
                .record-name {
                    font-size: 20px;
                }
                .record-price {
                    color: #999999;
                }
            }
            .record-sub {
                width: 30%;
                font-size: 12px;
                color: #aaa;
                line-height: 20px;
            }
        }
    }
}
</style>
