---
parent: Other
title: Komunalka
nav_order: 1
---

# Komunalka

| Тип					  | Номер			 | Посилання															   | Ключ                 | Оплата |
|:------------------------|:-----------------|:-----------------------------------------------------------------------:|:---------------------|--------|
| Укртелеком			  | 4600000020436017 | [EasyPay](https://easypay.ua/catalog/mobile/ukrtelecom)                 | UKRTELECOM           |        |
| Львівгаззбут			  | 0900503566	     | [EasyPay](https://easypay.ua/catalog/utility/lvov/lvovgaz)              | LVOVGAZ              |        |
| Львівводоканал		  | 8239870710	     | [EasyPay](https://easypay.ua/catalog/utility/lvov/vodokanal-lvov)       | VODOKANAL-LVOV       |        |
| Львівенергозбут		  | 1800550537	     | [EasyPay](https://easypay.ua/catalog/utility/lvov/lvovoblenergo)        | LVOVOBLENERGO        |        |
| Залізничне теплоенергія | 3200010903	     | [EasyPay](https://easypay.ua/catalog/utility/lvov/communal-lvov-merger) | COMMUNAL-LVOV-MERGER |        |
| Сигнівка Комуналка	  | 3050197027	     | [EasyPay](https://easypay.ua/catalog/utility/lvov/communal-lvov-merger) | COMMUNAL-LVOV-MERGER |        |


<script>

function getInitData() {
	return fetch("https://api.easypay.ua/api/system/createPage", {
		"headers": {
			"accept": "application/json, text/plain, */*",
			"appid": "e18ecdba-b592-4cba-bb3b-48486b0bfe49",
			"content-type": "application/json; charset=UTF-8",
			"partnerkey": "easypay-v2",
		},
		"body": null,
		"method": "POST",
	}).then(res => res.json());
}

async function getServiceData(row, initData, serviceKey, accountNumber) {
	const json = await fetch("https://api.easypay.ua/api/genericCommunalFlow/check", {
		"headers": {
			"accept":"application/json, text/plain, */*",
			"appid": initData.appId,
			"content-type":"application/json; charset=UTF-8",
			"pageid": initData.pageId,
			"partnerkey":"easypay-v2",
			"locale":"uk",
		},
		"body": JSON.stringify({
			"fields": [
				{
					"fieldName": "Account",
					"fieldValue": accountNumber
				}
			],
			"serviceKey":serviceKey,
			"amount":null,
			"multyCheckPaymentStepIndex":0,
		}),
		"method":"POST",
	}).then(res => res.json());

	return {
		row: row,
		serviceKey: serviceKey,
		accountInfo: json.accountInfo,
		products: json.products,
	}
}


getInitData().then(async (initData) => {
	const table = document.querySelector('table');
	const tbody = table.querySelector('tbody');
	const rows = [...tbody.querySelectorAll('tr')];

	Promise.all(rows.map(row => {
		const accountNumber = row.children[1].innerHTML;
		const serviceKey = row.children[3].innerHTML;
		const link = row.querySelector('a');
		link.href += '?account=' + accountNumber;
		return getServiceData(row, initData, serviceKey, accountNumber);
	})).then(datas => {
		return datas.map(data => {
			if (!data.products) {
				console.log('IGNORE', data);
				return undefined;
			}
			data.row.children[4].innerHTML = data.products.map(item => item.paymentAmount).join(', ') + " грн.";
			return data.products.reduce(
				(curr, item) => curr + item.paymentAmount,
				0
			);
		}).filter(item => item);
	}).then(ammounts => {
		const sum = ammounts.reduce((curr, item) => curr + item, 0);
		const sumRounded = Math.round(sum);
		console.log(sumRounded);
		const sumRow = document.createElement('tr');
		sumRow.innerHTML = `<th colspan="2"><h2>Сумма: </h2></th><th colspan="2"><h2>${sumRounded} грн.</h2></th>`;
		tbody.appendChild(sumRow);
	});
});

</script>
