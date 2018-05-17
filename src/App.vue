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
                        <img :src="item.link" onerror="this.src='../static/plane.jpg'" height="220" width="100%" />
                    </mu-card-media>
                    <mu-card-text>
                        <span>单价：{{item.price}}  </span>  <span class="unit">(NAS)</span> <br>
                        <span>库存：{{item.remained_quantity}} / {{item.total_quantity}}</span><br>
                        <span>卖家邮箱：{{item.seller_email}}</span>
                    </mu-card-text>
                    <mu-card-actions>
                        <mu-raised-button label="购买"  secondary @click.native="dialog = true;currItem = item;" />
                    </mu-card-actions>
                </mu-card>
            </div>
        </div>
        <div v-if="activeTab === 'tab2'" class="record scrollBar">
            <ul>
                <li v-for="(item,idx) in record" :key="idx">
                    <div class="record-icon">
                        <mu-icon value="card_giftcard" color="blue" :size="48" />
                    </div>
                    <div class="record-main">

                        <span class="record-name">{{item.name}} <br></span>
                        <span class="record-price">￥ {{item.price}} <span class="unit">(NAS)</span> <br></span>
                        
                    </div>
                    <div class="record-sub">
                        <span>数量： {{item.amount}} <span class="unit">(NAS)</span> </span> <br>
                        <span>总价：{{item.quantity}} <span class="unit">(NAS)</span> <br></span>
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
            <mu-text-field label="价格(NAS)" style="width: 30%" labelFloat v-model="price"/>
            <mu-text-field label="数量" style="width: 30%" labelFloat v-model="count"/>
            <br/>
            <mu-text-field  multiLine style="width: 90%" :rows="3" :rowsMax="6" label="描述"  labelFloat v-model="description" /> <br>
            <!-- <br/> -->
            <mu-text-field label="邮箱" labelFloat style="width: 30%" v-model="seller_email"/>
            <mu-text-field label="图片链接" labelFloat style="width: 30%" v-model="link"/><br/>
            <mu-flat-button slot="actions" @click="handleCloseAdd" primary label="取消"/>
            <mu-flat-button slot="actions" primary @click="handleAdd" label="确定"/>
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
const to = "n22T3E3aDorqs9aC92hTAQcLDpyyM93xds4";
neb.setRequest(new nebulas.HttpRequest("https://testnet.nebulas.io"));

neb.api.getNebState().then(res => {
  console.log(res, "state");
});

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
      currItem: {},
      myNebAddress: '',
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
      }
      if (value === "tab2") {
        this.getRecord();
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
          if (v.buyer === sessionStorage.getItem('myNebAddress')) {
            record.push(v);
          }
        });
        console.log(arr, "记录");
        this.record = record;
      });
    },
    handleTab2() {
        if(!sessionStorage.getItem('myNebAddress')) {
            alert('此地址暂无交易记录')
            this.activeTab = 'tab1'
        }
    },
    handleBuy() {
      const { buyCount, currItem: { id, price } } = this;
      const func = "buy";
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
            nebPay.queryPayInfo(this.serialNumber).then(res => {
              sessionStorage.setItem('myNebAddress',JSON.parse(res).data.from)
            });
            clearInterval(this.intervalQuery);
          }
        })
        .catch(function(err) {
          console.log(err);
        });
    }
  },
  created() {
    this.getItems();
  }
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
      height: 100px;
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
</style>
