---
cssclasses:
  - cards
---
```dataviewjs
// 获取 "物品" 文件夹下的所有页面
let pages = dv.pages('"物品"');

let totalPrice = 0;        // 总资产
let totalDailyPrice = 0;   // 日均价格总和
let currentTime = new Date(); // 当前时间
let totalItems = pages.length; // 总物品数
let retiredItems = 0;      // 已退役物品数
let activeItems = 0;       // 未退役物品数

// 遍历页面，计算每个页面的日均价格
for (let page of pages) {
    let price = page["购买价格"] || 0;   // 购买价格
    let purchaseDate = page["购买日期"] ? new Date(page["购买日期"]) : null; // 购买日期
    let isRetired = page["是否退役"] || false; // 是否退役字段

    // 统计退役和未退役物品数
    if (isRetired) {
        retiredItems += 1; // 如果「是否退役」为真，计为已退役
    } else {
        activeItems += 1; // 未退役物品
    }

    if (price > 0 && purchaseDate) {
        // 计算使用天数
        let daysUsed = isRetired
            ? (currentTime - purchaseDate) / (1000 * 60 * 60 * 24) // 假设退役日期为当前时间
            : (currentTime - purchaseDate) / (1000 * 60 * 60 * 24); // 当前时间 - 购买时间

        daysUsed = Math.max(1, Math.floor(daysUsed)); // 确保天数至少为1

        // 计算当前页面的日均价格
        let dailyPrice = price / daysUsed;

        // 累加数据
        totalPrice += price;         // 累加总资产
        totalDailyPrice += dailyPrice; // 累加所有日均价格
    }
}

// 定义HTML内容
const html = `
<div class="card-summary">
    <div class="card-header">
        我的资产
        <span class="sub-title">${activeItems}/${totalItems}</span> <!-- 显示未退役物品数/总物品数 -->
    </div>
    <div class="card-body">
        <div class="card-section">
            <div class="label">总资产</div>
            <div class="value">¥${totalPrice.toFixed(2)}</div>
        </div>
        <div class="divider"></div>
        <div class="card-section">
            <div class="label">总日均价格</div>
            <div class="value">¥${totalDailyPrice.toFixed(2)} 元/天</div>
        </div>
    </div>
</div>
`;

// 渲染HTML内容
dv.el("div", html);


```




```dataviewjs
const files = dv.pages('"物品"').where(f => f["购买价格"]);

// 构建卡片布局 HTML
let cardHTML = `<div class="card-container">`;

files.forEach(file => {
    const icon = file["icon"] || "📦";
    const name = file["旧物名称"] || "未命名";
    const price = file["购买价格"] || 0;

    // 判断是否退役（以是否退役勾选为准）
    const isRetired = file["是否退役"] || false;
    const retireDate = isRetired 
        ? '<span class="status retired-state">已退役</span>'
        : '<span class="status using-state">使用中</span>';

    // 购买日期与当前时间
    const buyDate = file["购买日期"] ? new Date(file["购买日期"]) : null;
    const today = new Date();

    // 计算使用时间
    let daysUsed = 0;
    if (buyDate) {
        const endDate = isRetired && file["退役日期"] ? new Date(file["退役日期"]) : today;
        daysUsed = Math.floor((endDate - buyDate) / (1000 * 60 * 60 * 24));
    }

    // 每天价格
    const dailyPrice = daysUsed > 0 ? (price / daysUsed).toFixed(2) + " 元/天" : "N/A";

    // 动态添加卡片样式类
    const cardClass = isRetired ? "card retired-card" : "card";

    // 添加卡片
    cardHTML += `
        <div class="${cardClass}">
            <div class="card-item card-icon">${icon}</div>
            <div class="card-item card-title">${name}</div>
            <div class="card-item">¥${price}</div>
            <div class="card-item card-status">${retireDate}</div>
            <div class="card-item">${daysUsed} 天</div>
            <div class="card-item">${dailyPrice}</div>
        </div>
    `;
});

cardHTML += `</div>`;

// 将卡片渲染到页面
dv.container.innerHTML = cardHTML;