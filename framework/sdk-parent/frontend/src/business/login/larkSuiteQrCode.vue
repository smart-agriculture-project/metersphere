<template>
  <div id="lark-suite-qr" class="lark-suite-qrName" />
</template>

<script>
import loadJs from "../../utils/remoteJs";
import {getLarkSuiteInfo} from "../../api/qrcode";

export default{
  name:'larkSuiteQrCode',
  data(){
    return{
    }
  },
  methods:{
    async initActive() {
      await getLarkSuiteInfo().then(res=> {
        const larkSuiteData = res.data;
        const redirectUrL = encodeURIComponent(window.location.origin);
        const url = `https://passport.larksuite.com/suite/passport/oauth/authorize?client_id=${larkSuiteData.agentId}&redirect_uri=${redirectUrL}&response_type=code&state=fit2cloud-lark-suite-qr`;

        const QRLoginObj = window.QRLogin({
          id: 'lark-suite-qr',
          goto: url,
          width: '300',
          height: '300',
          style: 'width:300px;height:300px;border-width: 0px;', // 可选的，二维码html标签的style属性
        });

        // function handleMessage
        if (typeof window.addEventListener !== 'undefined') {
          localStorage.setItem('loginType', 'LARK_SUITE');
          window.addEventListener('message', async (event) => {
            // 使用 matchOrigin 和 matchData 方法来判断 message 和来自的页面 url 是否合法
            if (QRLoginObj.matchOrigin(event.origin) && QRLoginObj.matchData(event.data)) {
              const loginTmpCode = event.data.tmp_code;
              // 在授权页面地址上拼接上参数 tmp_code，并跳转
              window.location.href = `${url}&tmp_code=${loginTmpCode}`;
            }
          });
        }
      });

    }
  },
  created() {
    loadJs('https://lf-package-us.larksuitecdn.com/obj/lark-static-us/lark/passport/qrcode/LarkSSOSDKWebQRCode-1.0.3.js').then(()=>{
      // 加载成功，进行后续操作
      this.initActive();
    })
  }
}

</script>

<style scoped>
  .lark-suite-qrName {

  }
</style>
