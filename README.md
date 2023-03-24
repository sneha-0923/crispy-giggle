# crispy-giggle
widget
<div class="shadowC">
  <center><h2><b>Power Consumption - Contractual PUE</b></h2></center>
  <!--  <div class="alert alert-info" ng-if="!$root.tableLoader">
                  <fa fa-4x name="spinner" spin="true"></fa> ${Loading data}...
               <i class="fa fa-spinner fa-spin" style="font-size:50px;margin-left: 40%;"></i>${Loading data}...
                    </div> -->
  <div class="tableData" id="style-scroll">
    <table class="table table-bordered">
      <thead style="font-style:calibri;font-size:16px;">
        <tr>
          <th rowspan="2">GEN</th>
          <th rowspan="2">Supplier Site Name</th>
          <th rowspan="2">OCI Site</th>
          <th rowspan="2">DH/Cage</th>
          <th rowspan="2">Contractual PUE</th>
          <th ng-repeat="month in data.finalarr" colspan="6">{{month}}</th>
        </tr>

        <tr>
          <th ng-repeat-start="month in data.finalarr">Lower IT Load (kW)</th>
          <th>Average IT Load (kW)</th>
          <th>Peak IT Load (kW)</th>
          <th>Total Contracted Load (kW)</th>
          <th>Load%</th>
          <th ng-repeat-end>Contractual PUE(As per table)</th> 
        </tr>

      </thead>


      <tr ng-repeat="(keystr,consumption_data) in data.consumption">
        <td>{{consumption_data.value.gen}}</td>
        <td>{{consumption_data.value.supplier_site_name}}</td>
        <td>{{consumption_data.value.oci_site}}</td>
        <td>{{consumption_data.value.dh_cage}}</td>
        <td>{{consumption_data.value.contractual_pue}}</td>
        <td ng-repeat="month in data.finalarr">{{consumption_data[month].lower_it_load_kw}}</td>
        <td ng-repeat="month in data.finalarr">{{consumption_data[month].average_it_load_kw}}</td>
        <td ng-repeat="month in data.finalarr">{{consumption_data[month].peak_it_load_kw}}</td>
        <td ng-repeat="month in data.finalarr">{{consumption_data[month].total_contracted_load_kw}}</td>
        <td ng-repeat="month in data.finalarr">{{consumption_data[month].load}}</td>
        <td ng-repeat="month in data.finalarr">{{consumption_data[month].contractual_pue_as_per_table}}</td>
      </tr>

    </table>
  </div>
</div>


(function() {
	/* populate the 'data' object */
	/* e.g., data.table = $sp.getValue('table'); */

	var quarter ="Q3";
	data.month ="ALL";
	data.monthsToDisplay = [];
	data.finalarr = [];
	data.monthsToDisplay.push(data.month);

	if(data.month == "ALL"){
		if(quarter == 'Q1') {
			data.monthsToDisplay = ["APR","MAY","JUN"];
		}
		else if(quarter == 'Q2') {
			data.monthsToDisplay = ["JUL","AUG","SEP"];
		}
		else if(quarter == 'Q3') {
			data.monthsToDisplay = ["OCT","NOV","DEC"];
		}
		else if(quarter == 'Q4') {
			data.monthsToDisplay = ["JAN","FEB","MAR"];
		}


		//data.monthsToDisplay.splice(0,0,data.quarter);
	}/*else{
			data.monthsToDisplay.push(data.month);
		}*/

	//gs.addInfoMessage("data:"+JSON.stringify(data.monthsToDisplay));
	/*	switch(quarter){
			case "Q1":
				year = input.ChangedQuarter.split(" ")[1].split('-')[0];
				break;
			case "Q2":
				year = input.ChangedQuarter.split(" ")[1].split('-')[0];
				break;
			case "Q3":
				year = input.ChangedQuarter.split(" ")[1].split('-')[0];
				break;
			case "Q4":
				year = input.ChangedQuarter.split(" ")[1].split('-')[1];
				break;
		}*/

	//year = '2023';

	var property  = new GlideRecord('x_sitl_goinfinit_goinfinit_properties');
	property.addQuery('key','Months'); 
	property.query();
	if(property.next()) {
		var monthData = JSON.parse(property.getValue('value'));
	}
	var nowDate = new Date();
	var nowMonthNumber = nowDate.getUTCMonth()+1;

	for(var i = 0; i < data.monthsToDisplay.length; i++) {
		var monthNameValue = data.monthsToDisplay[i];
		//gs.addInfoMessage(monthNameValue);
		var monthDataValue = monthData[monthNameValue].MonthValue;
		//gs.addInfoMessage(monthDataValue);
		//gs.addInfoMessage("monthNameValue: " + monthNameValue + ", monthDataValue: " + monthDataValue + ", nowMonthNumber: " + nowMonthNumber);
		if(monthDataValue != nowMonthNumber) {
			var monthNameValuee = monthData[monthNameValue].Monthname;
			//gs.addInfoMessage("i  "+data.monthsToDisplay[i]);
			data.finalarr.push(data.monthsToDisplay[i]); 
		}
		else{
			break;
		}
	}

	//gs.addInfoMessage("finalarr: " + data.finalarr);

	data.consumption=[];
	//gs.addInfoMessage("data.monthsToDisplay   "+data.monthsToDisplay);
	var power = new GlideRecord('x_sitl_goinfinit_pc_contractual_pue');
	power.addEncodedQuery('monthIN'+data.finalarr);
	//power.addQuery('year',year);
	power.query();
	var consumption_data={};
	while(power.next())
	{
		var gen= power.getValue('gen');
		var supplier_site_name = power.getDisplayValue('supplier_site_name');
		var oci_site = power.getDisplayValue('oci_site'); 
		var dh_cage = power.getDisplayValue('dh_cage'); 
		var contractual_pue = power.getDisplayValue('contractual_pue');
		var keystr = gen+"-"+supplier_site_name+"-"+oci_site+"-"+dh_cage+"-"+contractual_pue; 
		if(!consumption_data[keystr])
		{
			consumption_data[keystr] = {};
			consumption_data[keystr]['value'] = {};
			consumption_data[keystr]['value']['gen'] = gen;
			consumption_data[keystr]['value']['supplier_site_name'] = supplier_site_name;
			consumption_data[keystr]['value']['oci_site'] = oci_site;
			consumption_data[keystr]['value']['dh_cage'] = dh_cage;
			consumption_data[keystr]['value']['contractual_pue'] = contractual_pue;

		}
		var month = power.getDisplayValue('month');
		while(!consumption_data[keystr][month]){
			consumption_data[keystr][month] = {};
		}
		consumption_data[keystr][month]['lower_it_load_kw'] =  power.getDisplayValue('lower_it_load_kw');
		consumption_data[keystr][month]['average_it_load_kw'] =  power.getDisplayValue('average_it_load_kw');
		consumption_data[keystr][month]['peak_it_load_kw'] =  power.getDisplayValue('peak_it_load_kw');
		consumption_data[keystr][month]['total_contracted_load_kw'] =  power.getDisplayValue('total_contracted_load_kw');
		consumption_data[keystr][month]['load'] =  power.getDisplayValue('load');
		consumption_data[keystr][month]['contractual_pue_as_per_table'] =  power.getDisplayValue('contractual_pue_as_per_table');
		data.consumption.push(consumption_data[keystr][month]);
	}	
	
	

	gs.addInfoMessage(JSON.stringify(data.consumption));
	
	/*var monthlyData = {};
	for (var i = 0; i <data.finalarr.length; i++) {
		var month = data.finalarr[i];
		var monthData = {
			"Lower IT Load (kW)": power.getDisplayValue('lower_it_load_kw'),
			"Average IT Load (kW)": power.getDisplayValue('average_it_load_kw'),
			"Peak IT Load (kW)": power.getDisplayValue('peak_it_load_kw'),
			"Total Contracted Load (kW)": power.getDisplayValue('total_contracted_load_kw'),
			"Load%": power.getDisplayValue('load'),
			"Contractual PUE(As per table)": power.getDisplayValue('contractual_pue_as_per_table')
		};
		monthlyData[month] = monthData;
	}

	gs.addInfoMessage(JSON.stringify(monthlyData));

	//gs.addInfoMessage("data:"+JSON.stringify(data.monthsToDisplay));
	//gs.addInfoMessage("data:"+JSON.stringify(data.consumption));*/

})();
