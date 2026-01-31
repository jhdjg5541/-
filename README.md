# -
现实的金币买格子玩
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>付费格子网站-专属格子认购平台</title>
    <style>
        * {margin: 0;padding: 0;box-sizing: border-box;}
        body {font-family: 微软雅黑;background: #f5f5f5;padding: 20px 0;}
        /* 顶部导航 */
        .header {text-align: center;padding: 15px;background: #fff;border-bottom: 2px solid #333;margin-bottom: 20px;}
        .header h1 {color: #222;margin-bottom: 10px;}
        .header p {color: #666;font-size: 14px;}
        /* 格子容器 核心！可改格子数量和大小 */
        .grid-box {width: 90%;max-width: 1200px;margin: 0 auto;display: grid;
                   /* 10列布局，可改repeat(10,1fr)为5列就是repeat(5,1fr)，1fr自动均分 */
                   grid-template-columns: repeat(10,1fr);
                   gap: 5px; /* 格子间距 */
        }
        /* 格子样式 */
        .grid-item {
            border: 2px solid #333;
            border-radius: 4px;
            padding: 8px 0;
            text-align: center;
            font-size: 12px;
            cursor: pointer;
            transition: all 0.3s;
            aspect-ratio: 1/1; /* 保证格子正方形 */
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }
        /* 未售格子 */
        .item-unused {background: #e8f5e9;color: #2e7d32;}
        .item-unused:hover {background: #c8e6c9;transform: scale(1.02);}
        /* 已售格子 */
        .item-sold {background: #ffebee;color: #c62828;border-color: #c62828;cursor: not-allowed;}
        /* 锁定格子 */
        .item-lock {background: #fff3e0;color: #ef6c00;border-color: #ef6c00;cursor: not-allowed;}
        /* 弹窗样式 购买弹窗 */
        .modal {
            position: fixed;
            top: 0;left: 0;right: 0;bottom: 0;
            background: rgba(0,0,0,0.7);
            display: none;
            justify-content: center;
            align-items: center;
            z-index: 999;
        }
        .modal-content {
            background: #fff;
            width: 90%;max-width: 400px;
            padding: 25px;
            border-radius: 8px;
            text-align: center;
        }
        .modal h3 {margin-bottom: 15px;color: #333;}
        .price-text {color: #e53e3e;font-size: 24px;font-weight: bold;margin: 10px 0;}
        .pay-btn {
            background: #4CAF50;
            color: white;
            border: none;
            padding: 12px 30px;
            border-radius: 5px;
            font-size: 16px;
            cursor: pointer;
            margin: 10px 5px;
        }
        .close-btn {background: #9e9e9e;}
        .pay-img {width: 180px;height: 180px;margin: 15px auto;display: none;}
        .tips {font-size: 12px;color: #666;margin-top: 15px;line-height: 1.5;}
        /* 底部说明 */
        .footer {width: 90%;max-width: 1200px;margin: 30px auto;text-align: center;font-size: 13px;color: #666;}
        .footer p {margin: 5px 0;}
        /* 手机适配，小屏自动变5列 */
        @media (max-width:768px) {
            .grid-box {grid-template-columns: repeat(5,1fr);}
            .grid-item {font-size: 10px;padding: 5px 0;}
        }
    </style>
</head>
<body>
    <!-- 顶部 -->
    <div class="header">
        <h1>专属格子认购平台</h1>
        <p>每格独立产权，认购后永久展示，点击格子即可下单</p >
    </div>

    <!-- 核心格子区域 10x10=100格，可增减 -->
    <div class="grid-box" id="gridContainer">
        <!-- 格子自动生成，无需手动写，JS自动渲染100格 -->
    </div>

    <!-- 购买弹窗 -->
    <div class="modal" id="buyModal">
        <div class="modal-content">
            <h3>认购格子 <span id="gridNum">NO.0</span></h3>
            <div class="price-text">￥<span id="gridPrice">2</span> 元/永久</div>
            <button class="pay-btn" id="showPayBtn">立即支付</button>
            <button class="pay-btn close-btn" onclick="closeModal()">取消</button>
            <!-- 收款码，替换成自己的微信/支付宝收款码图片 -->
            < img src="微信收款码.jpg" alt="微信收款" class="pay-img" id="wxPay">
            < img src="支付宝收款码.jpg" alt="支付宝收款" class="pay-img" id="zfbPay">
            <div class="tips">
                1. 支付后截图凭证，联系客服核验<br>
                2. 核验通过后，格子标记已售并展示您的昵称<br>
                3. 格子一经售出，概不退换
            </div>
        </div>
    </div>

    <!-- 底部 -->
    <div class="footer">
        <p>客服微信：XXXXXXX（替换成自己的联系方式）</p >
        <p>购买须知：每格永久使用权，支持自定义展示昵称，禁止违规内容</p >
        <p>© 2025 专属格子网 版权所有</p >
    </div>

    <script>
        // 核心配置，直接修改这里即可，不用动其他代码
        const config = {
            gridCount: 100, // 总格子数，默认100格
            singlePrice: 2, // 每格价格，默认2元，可自定义
            soldList: [3,7,12,18,25], // 已售格子编号，填数字即可，比如[1,5,8]就是1/5/8格已售
            lockList: [5,9], // 锁定格子编号（下单未支付）
            nickName: "已售" // 已售格子默认显示文字，可改买家昵称
        }

        // 1. 初始化格子
        const gridContainer = document.getElementById('gridContainer');
        for(let i=1; i<=config.gridCount; i++){
            const grid = document.createElement('div');
            grid.className = 'grid-item item-unused';
            grid.id = `grid${i}`;
            grid.innerHTML = `NO.${i}`;
            // 标记已售/锁定格子
            if(config.soldList.includes(i)){
                grid.className = 'grid-item item-sold';
                grid.innerHTML = `${config.nickName}`;
            }else if(config.lockList.includes(i)){
                grid.className = 'grid-item item-lock';
                grid.innerHTML = `锁定中`;
            }
            // 未售格子绑定点击事件
            if(!config.soldList.includes(i) && !config.lockList.includes(i)){
                grid.onclick = () => openModal(i);
            }
            gridContainer.appendChild(grid);
        }

        // 2. 打开购买弹窗
        const buyModal = document.getElementById('buyModal');
        const gridNum = document.getElementById('gridNum');
        const gridPrice = document.getElementById('gridPrice');
        function openModal(index){
            gridNum.innerText = `NO.${index}`;
            gridPrice.innerText = config.singlePrice;
            buyModal.style.display = 'flex';
            // 打开弹窗隐藏收款码
            document.getElementById('wxPay').style.display = 'none';
            document.getElementById('zfbPay').style.display = 'none';
        }

        // 3. 关闭弹窗
        function closeModal(){
            buyModal.style.display = 'none';
        }

        // 4. 显示收款码
        document.getElementById('showPayBtn').onclick = function(){
            this.style.display = 'none';
            document.getElementById('wxPay').style.display = 'block';
            document.getElementById('zfbPay').style.display = 'block';
        }

        // 点击弹窗外层关闭
        buyModal.onclick = function(e){
            if(e.target === this) closeModal();
        }
    </script>
</body>
</html>
