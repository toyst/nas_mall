<template>
	<div id="app">
        <mu-tabs :value="activeTab" @change="handleTabChange">
            <mu-tab value="tab1" icon="list" title="商品列表"/>
            <mu-tab value="tab2" icon="history" title="购买记录" @active="handleTab2" />
        </mu-tabs>
        <div v-if="activeTab === 'tab1'" class="goods-list scrollBar">
            
            <div v-for="item in items" :key="item.id">
                <mu-card>
                    <mu-card-header :title="item.name">
                    </mu-card-header>
                    <mu-card-media :subTitle="item.description">
                        <img :src="item.link" onerror="this.src='https://nebulas.io/docs/NebulasLogo.svg'" height="220" width="100%" />
                    </mu-card-media>
                    <mu-card-text>
                        <span>单价：{{item.price}}  </span>  <span class="unit">(NAS)</span> <br>
                        <span>库存：{{item.remained_quantity}} / {{item.total_quantity}}</span><br>
                        <span style="word-break: break-all">卖家邮箱：{{item.seller_email}}</span> <br>
                        <span style="word-break: break-all">id： {{item.id}}</span><br>
                        <span style="word-break: break-all">卖家id：{{item.seller}}</span>

                    </mu-card-text>
                    <mu-card-actions>
                        <mu-raised-button label="购买"  secondary @click.native="dialog = true;currItem = item;" />
                    </mu-card-actions>
                </mu-card>
            </div>
        </div>
        <div v-if="activeTab === 'tab2'" class="record scrollBar">
           <span v-if="record.length === 0">暂无购买记录，快去购买商品吧~</span>
            <ul>
                <li v-for="(item,idx) in record" :key="idx">
                    <div class="record-icon">
                        <mu-icon value="card_giftcard" color="blue" :size="48" />
                    </div>
                    <div class="record-main">

                        <span class="record-name">{{item.name}} <br></span>
                        
                        <span class="record-price">￥ {{item.price}} <span class="unit">(NAS)</span> <br></span>
                        <span>总价：{{item.amount}} <span class="unit">(NAS)</span> <br></span>
                        <span style="word-break: break-all"><br>
                          id:  {{item.commodity_id}}
                        </span><br>
                        <span style="word-break: break-all">
                          交易id： {{item.id}}
                        </span>
                    </div>
                    <div class="record-sub">
                        <span style="word-break: break-all">买家： {{item.buyer}}</span><br>
                        <span>数量： {{item.quantity}} <span class="unit"></span> </span> <br>
                        
                        <span>时间： {{new Date(item.timestamp * 1000).toLocaleString()}}</span>
                    </div>
                </li>
            </ul>
        </div>
        <mu-dialog :open="dialog" title="输入购买数量" @close="handleCloseBuy">
            <mu-text-field hintText="请输入整数" v-model="buyCount" />
            <mu-flat-button slot="actions" @click="handleCloseBuy" primary label="取消"/>
            <mu-flat-button slot="actions" primary @click="handleBuy" label="确定"/>
        </mu-dialog>
        <mu-dialog :open="addDialog" title="发布商品" @close="handleCloseAdd">
            <mu-text-field label="名称" style="width: 30%" labelFloat v-model="name"/> 
            <mu-text-field label="价格(NAS)" hintText="请输入数字" style="width: 30%" labelFloat v-model="price"/>
            <mu-text-field label="数量" hintText="请输入整数" style="width: 30%" labelFloat v-model="count"/>
            <br/>
            <mu-text-field  style="width: 90%" hintText="最多不超过50个字符" multiLine :rows="2" :rowsMax="3" @textOverflow="handleInputOverflow" :maxLength="50" :errorText="inputErrorText"  label="描述"  labelFloat v-model="description" /> <br>
            <!-- <br/> -->
            <mu-text-field label="邮箱" labelFloat style="width: 30%" v-model="seller_email"/><br>

            <mu-text-field label="图片链接(若链接无效则展示默认图片)" hintText="图片上传功能正在开发中，请手动输入链接" labelFloat style="width: 90%" v-model="link"/><br/>
            <mu-flat-button slot="actions" @click="handleCloseAdd" primary label="取消"/>
            <mu-flat-button slot="actions" primary @click="handleAdd" label="确定"/>
        </mu-dialog>
        <mu-dialog :open="dialogAddress" title="请输入钱包地址">
            <div class="historys">
              <div v-for="(item,idx) in historys" :key="idx" @click="nebAddress = item">
                {{item}}
              </div> 
            </div>
            <mu-text-field hintText="请输入合法的钱包地址" fullWidth v-model="nebAddress" />
            <mu-flat-button slot="actions" @click="handleCloseAddress" primary label="取消"/>
            <mu-flat-button slot="actions" primary @click="handleAddress" label="确定" :disabled="nebAddress.trim() === ''" />
        </mu-dialog>
        <mu-dialog :open="dialogIntr" title="使用说明">
            <p class="intr">
              欢迎来到NAS商品交易平台，本平台是一个基于星云链并用星云币支付的商品交易DAPP，主要功能有发布商品、商品列表、商品购买和交易记录查询。<br>
              由于区块链钱包的特殊性，您需要在谷歌浏览器上打开此页面并且提前安装好 <a href="https://github.com/ChengOrangeJu/WebExtensionWallet" target="_blank">谷歌扩展钱包</a>  才可正常体验，谢谢！
              <br>
              一、发布商品<br>
              点击页面中的加号按钮，按格式要求正确的填写您的商品信息<br>
              (暂不支持图片上传，请手动输入图片链接，若链接无效则展示默认图片)。<br>
              二、商品列表<br>
              页面会自动展示所有的商品，刷新浏览器可获取最新的商品信息。<br>
              三、商品购买<br>
              您可以挑选中意的商品并通过卖家的联系方式及时沟通，确定购买之后输入所需数量，确保商品有足够库存并且您有足够的nas余额即可购买。<br>
              四、交易记录查询<br>
              输入您的nas钱包地址，确保有购买经历并且地址合法即可查看您的所有购买记录。
            </p>
            <mu-flat-button slot="actions" @click="dialogIntr = false" primary label="知道了"/>
        </mu-dialog>
        <mu-float-button icon="add" primary  style="position: fixed;bottom: 35px; right: 31vw" @click="addDialog = true" />
	</div>
</template>

<script>
import NebPay from "nebPay";
import nebulas from "nebulas";
// import defaultImg from '../static/plane.jpg'
const nebPay = new NebPay();
const Account = nebulas.Account;
const neb = new nebulas.Neb();
const from = Account.NewAccount().getAddressString();
const to = "n1qQfgZG5aGUM5WYne4dwfcFqBC2hV4hFLz";
neb.setRequest(new nebulas.HttpRequest("https://testnet.nebulas.io"));

neb.api.getNebState().then(res => {
  console.log(res, "state");
});

const safeJSON = (json, defaultValue) => {
    try {
        return JSON.parse(json)
    } catch (e) {
        return defaultValue || json
    }
}
const reg = new RegExp("^[a-z0-9]+([._\\-]*[a-z0-9])*@([a-z0-9]+[-a-z0-9]*[a-z0-9]+.){1,63}[a-z0-9]+$");
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
const CheckImgExists = imgurl => {
  var ImgObj = new Image(); //判断图片是否存在
  ImgObj.src = imgurl;
  //没有图片，则返回-1
  console.log(
    ImgObj.fileSize,
    ImgObj.width,
    ImgObj.height,
    ImgObj.offsetHeight
  );
  if (ImgObj.fileSize > 0 || (ImgObj.width > 0 && ImgObj.height > 0)) {
    return true;
  } else {
    return false;
  }
};
export default {
  name: "App",
  data() {
    return {
      historys: safeJSON(localStorage.getItem('myNebAddress'),[]),
      dialogIntr: true,
      inputErrorText: "",
      nebAddress: "n1S6RQy6F6GetEU6hCyu8enfS1kWZVkfYSf",
      dialogAddress: false,
      currItem: {},
      // defaultImg,
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
      record: [],
      serialNumber: "",
      intervalQuery: ""
    };
  },
  watch: {
    activeTab(value) {
      if (value === "tab1") {
        this.getItems();
        this.record = []
      }
    }
  },
  methods: {
    handleAdd() {
      const { name, description, price, seller_email, count, link } = this;
      if(name.trim === '') {
        alert('请输入正确的名称')
        return
      }
      if(description.length > 50) {
        alert('商品描述在50字以内')
        return
      }
      if(isNaN(Number(price)) || price <= 0 ) {
        alert('请输入正确的价格')
        return
      }
      if(isNaN(Number(count)) || Number.isInteger(count) || count < 1) {
        alert('数量必须为正整数')
        return
      }
      if(!reg.test(seller_email)) {
        alert('请输入正确的邮箱')
        return
      }
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
      const _this = this;
      const options = {
        listener: function(res) {
          console.log("交易返回信息", res);
          this.get;
        },
        callback: undefined //可以指定交易查询服务器
      };
      this.serialNumber = nebPay.call(
        to,
        value,
        func,
        addCommodityArgs,
        options
      );
      this.intervalQuery = setInterval(function() {
        _this.funcIntervalQuery();
      }, 10000);
      this.handleCloseAdd();
    },
    getItems() {
      call({
        func: "getCommodities",
        callArgs: getParams([""])
      }).then(res => {
        let items = JSON.parse(res.result);
        console.log(items, "商品列表");
        this.items = items;
      });
    },
    getRecord() {
      call({
        func: "getTxes",
        callArgs: getParams([""])
      }).then(res => {
        const arr = JSON.parse(res.result);
        const record = [];
        arr.forEach(v => {
          if (v.buyer === this.nebAddress) {
            record.push(v);
          }
        });
        console.log(arr, "记录");
        this.record = record;
        this.nebAddress = ''
      });
    },
    handleTab2() {
      this.dialogAddress = true;
    },
    handleBuy() {
      let { buyCount, currItem: { id, price, remained_quantity} } = this;
      buyCount = Number(buyCount)
      price = Number(price)
      remained_quantity = Number(remained_quantity)
      const func = "buy";

      if (isNaN(buyCount) || buyCount <= 0 || buyCount > remained_quantity ) {
        alert('购买数量必须为小于库存的正数')
        this.buyCount = 0
        return
      }
      const value = String(price * buyCount + 0.01);
      const args = getParams([id, buyCount]);
      const options = {
        listener: function(res) {
          console.log("交易返回信息", res);
        },
        callback: undefined //可以指定交易查询服务器
      };
      this.serialNumber = nebPay.call(to, value, func, args, options);

      this.intervalQuery = setInterval(() => {
        this.funcIntervalQuery();
      }, 10000);
      this.handleCloseBuy();
    },
    handleCloseBuy() {
      this.dialog = false;
      this.buyCount = 0;
      this.currItem = {};
    },
    handleAddress() {
      if( !Account.isValidAddress(this.nebAddress)) {
        alert('该地址无效，请重新输入')
        this.nebAddress = ''
        return
      }
      
      const history = [...new Set([...safeJSON(localStorage.getItem('myNebAddress'),[]),this.nebAddress])]
      localStorage.setItem("myNebAddress", JSON.stringify(history))
      this.historys = history
      this.getRecord()  
      this.handleCloseAddress();
    },
    handleCloseAddress() {
      if (!this.nebAddress) {
        this.activeTab = "tab1";
      }
      this.dialogAddress = false;
    },
    handleTabChange(val) {
      this.activeTab = val;
    },
    handleCloseAdd() {
      this.addDialog = false;
      this.name = this.description = this.price = this.seller_email = this.count = this.link =
        "";
    },
    funcIntervalQuery() {
      nebPay
        .queryPayInfo(this.serialNumber) //search transaction result from server (result upload to server by app)
        .then(resp => {
          console.log("tx result: " + resp); //resp is a JSON string
          var respObject = JSON.parse(resp);
          if (respObject.code === 0) {
            alert("操作成功，请刷新页面");
            clearInterval(this.intervalQuery);
          }
        })
        .catch(function(err) {
          console.log(err);
        });
    },
    handleInputOverflow(isOverflow) {
      this.inputErrorText = isOverflow ? "超过啦！！！！" : "";
    }
  },
  created() {
    if(localStorage.getItem('myNebAddress') === null) {
      localStorage.setItem('myNebAddress',JSON.stringify([]))
    }
    this.getItems();
  }
};
</script>

<style scoped lang="scss">
#app {
  min-height: 100vh;
  width: 50vw;
  margin: 0 auto;
  background: white;
}
.goods-list {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  padding: 20px;
  height: calc(100vh - 80px);
  overflow: auto;
  > div {
    width: 48%;
    margin-bottom: 20px;
  }
}
.record {
  width: 100%;
  padding: 20px;
  height: calc(100vh - 80px);
  overflow: auto;
  > ul {
    width: 100%;
    > li {
      list-style: none;
      width: 100%;
      padding: 0 20px;
      height: 200px;
      box-shadow: 0 0 24px 0 rgba(15, 66, 76, 0.25);
      display: flex;
      margin-bottom: 30px;
      justify-content: space-between;
      align-items: center;
      transition: all 0.2s;
      &:hover {
        transform: translateY(-5px);
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
.unit {
  color: #bbb;
  margin-left: 5px;
  font-size: 12px;
}
.scrollBar {
  &::-webkit-scrollbar-thumb {
    background-color: #ddd;
    border-radius: 20px;
    z-index: 999;
    cursor: pointer;
    &:hover {
      background-color: rgba(136, 136, 136, 0.5);
    }
  }
  &::-webkit-scrollbar {
    height: 6px;
    width: 6px;
  }
  &::-webkit-scrollbar-track {
    border-radius: 20px;
  }
}
.intr {
  font-size: 14px;
  line-height: 35px;
}
.historys {

  >div {
    margin-bottom: 5px;
    background: #eee;
    padding: 5px 10px;
    font-size: 12px;
    width: 255px;
    cursor: pointer;
    // color: white;
    &:hover {
      color: rgba(255,255,255,0.8);
    }
  }
}
</style>
