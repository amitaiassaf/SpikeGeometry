# GeometryGlycoproteinEvolution
The repository is related to the manuscript [**Viral surface geometry shapes the influenza and coronavirus spike evolution**]


<!DOCTYPE html>
<html>

<head>
	<title>Corona Virus Trend</title>
	<!-- Firebase JS SDK -->
	<script src="https://www.gstatic.com/firebasejs/7.24.0/firebase-app.js"></script>
	<script src="https://www.gstatic.com/firebasejs/7.6.1/firebase-auth.js"></script>
	<script src="https://www.gstatic.com/firebasejs/7.6.1/firebase-database.js"></script>
	<script src="https://www.gstatic.com/firebasejs/7.24.0/firebase-analytics.js"></script>

	<!-- Date pickers -->
	<link href="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/3.3.7/css/bootstrap.css" rel="stylesheet" />
	<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/2.1.3/jquery.js"></script>
	<link href="https://cdnjs.cloudflare.com/ajax/libs/bootstrap-datepicker/1.7.1/css/bootstrap-datepicker3.css"
		rel="stylesheet" />
	<script src="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/3.3.7/js/bootstrap.js"></script>
	<script src="https://cdnjs.cloudflare.com/ajax/libs/bootstrap-datepicker/1.7.1/js/bootstrap-datepicker.js"></script>

	<!-- Graphs with Plotly !-->
	<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>

	<script>
		// Firebase configuration
		var firebaseConfig = {
			apiKey: "AIzaSyD38gEAhiQxAcoKz7_ej_qfFuNVKBM9APg",
			authDomain: "corona-trend.firebaseapp.com",
			databaseURL: "https://corona-trend.firebaseio.com",
			projectId: "corona-trend",
			storageBucket: "corona-trend.appspot.com",
			messagingSenderId: "263891468811",
			appId: "1:263891468811:web:2c7f51c89677f0e5eb7d0e",
			measurementId: "G-FW0ZTKZSLB"
		};
		// Initialize Firebase
		firebase.initializeApp(firebaseConfig);
		firebase.analytics();
	</script>
</head>

<body>
	<center><h1>Spike evolution of SARS-CoV-2</h1></center>
	<div><b>Histogram:</b> The histogram depicts the sequence entropy (see eq.(1) in the manuscript) of each spike residue computed for the SARS-CoV-2 spike. Sequences collect in the time period that was selected. (Sequences taken from www.gisaid.org). <br>
		<b>Scatter plot:</b> Scatter plot of the epitope clusters entropy of sequences collected at the selected time period, versus.the epitope cluster affinity estimated from the simulations. The correlation coefficient between them is shown above the plot. Clusters that contain residues belonging to the Receptor-Binding-Domain (RBD) are in green and those containing residues belonging to the Receptor-Binding-Motif (RBM) are in red. (The number of clusters is 60).</div>
	<input type="text" class="form-control" id="datepicker">
	<button id='gobutton'> Change start month </button>
	<input type="text" class="form-control" id="datepickerEnd">
	<button id='gobuttonEnd'> Change end month </button>
	<div id='hist'> </div>
	<div id='cor'> </div>
	<div id='scat'> </div>
	<script>
		var datePickerElement = $("#datepicker").datepicker({
			format: "mm-yyyy",
			viewMode: "months",
			minViewMode: "months",
			startDate: new Date(2019, 11, 1),
			endDate: new Date(2020, 11, 1)
		});
		var datePickerElementEnd = $("#datepickerEnd").datepicker({
			format: "mm-yyyy",
			viewMode: "months",
			minViewMode: "months",
			startDate: new Date(2019, 11, 1),
			endDate: new Date(2020, 11, 1)
		});
		function histogram(data) {
			if (data) {
				var yValues = [];
				var xValues = [];
				for (var i = 0; i < data.length; i++) {
					yValues[i] = data[i].y;
					xValues[i] = data[i].x;
				}
				var dataBar = {
					x: xValues,
					y: yValues,
					type: 'bar'
				};
				var layout = {
                title: 'Sequence entropy',
                xaxis: {
                  title: 'S residue sequence index',
                  showgrid: false,
                  zeroline: false
                },
                yaxis: {
                  title: 'Residue entropy (mutability)',
                  showline: true
                }
              	};
				Plotly.newPlot('hist', [dataBar],layout);
			}
		}
		function scatter(data) {
			if (data) {
				var cluster = {
					x: [],
					y: [],
					mode: 'markers',
					type: 'scatter',
					marker: {
						color: 'rgb(0, 0, 0)' // black
					},
					name: 'cluster'
				};
				var RBD = {
					x: [],
					y: [],
					mode: 'markers',
					type: 'scatter',
					name: 'RBD',
					marker: {
						color: 'rgb(51, 204, 51)' // green
					}
				};
				var RBM = {
					x: [],
					y: [],
					mode: 'markers',
					type: 'scatter',
					name: 'RBM',
					marker: {
						color: 'rgb(255, 0, 0)'	// red
					}
				};
				var scatterData = data[0]; // adjust to contract glitch
				for (var i = 0; i < scatterData.length; i++) {
					if (scatterData[i].y == 0) {
						continue; // log(y)
					}
					if (scatterData[i].color == 'black') {
						cluster.x.push(scatterData[i].x);
						cluster.y.push(scatterData[i].y);
					} else if (scatterData[i].color == 'red') {
						RBM.x.push(scatterData[i].x);
						RBM.y.push(scatterData[i].y);
					} else {
						RBD.x.push(scatterData[i].x);
						RBD.y.push(scatterData[i].y);
					}
				}
				var dataScat = [cluster, RBD, RBM];
              				
              	var layout = {
                title: 'Antibody pressure vs. mutability for SARS-CoV-2 spike (S protein)',
                xaxis: {
                  title: 'Antibody pressure on residues from computational model',
                  showgrid: false,
                  zeroline: false
                },
                yaxis: {
                  title: 'Residues mutability estimated from sequences',
                  showline: true
                }
              	};
				
				Plotly.newPlot('scat', dataScat,layout);
			}
		}
		function setCorrelationLabel(data) {
			if (data) {
				$('#cor').text(`Correlation coefficient: ${data}`);
				//$('#cor').style.fontSize = 'xx-large';
			}
		}
		function getRefId(dateStr) {
			if (dateStr !== '') {
				var month = dateStr.split('-')[0];
				var year = dateStr.split('-')[1].slice(-2);
				var isJanuary = month == '01' ? true : false;
				var prevMonth = isJanuary ? '12' : month - 1;
				if (prevMonth.toString().length == 1) { prevMonth = `0${prevMonth}` }
				var prevYear = isJanuary ? year - 1 : year;
				return `D${prevYear}${prevMonth}01${year}${month}01`;
			}
		}
		function getRefIdStartEnd(dateStrStart,dateStrEnd) {
			if (dateStrStart !== '') {
				var Stmonth = dateStrStart.split('-')[0];
				var Styear = dateStrStart.split('-')[1].slice(-2);
				var isJanuary = Stmonth == '01' ? true : false;

				var Endmonth = dateStrEnd.split('-')[0];
				var Endyear = dateStrEnd.split('-')[1].slice(-2);
				var isJanuary = Endmonth == '01' ? true : false;

				//var prevMonth = isJanuary ? '12' : month - 1;
				//if (prevMonth.toString().length == 1) { prevMonth = `0${prevMonth}` }
				//var prevYear = isJanuary ? year - 1 : year;
				//return `D${prevYear}${prevMonth}01${year}${month}01`;
				return `D${Styear}${Stmonth}01${Endyear}${Endmonth}01`;
			}
		}
		function paintWithRefId(refId) {
			firebase.database().ref(refId).once('value').then(function (snapshot) {
				var data = snapshot.val();
				console.log(data);
				if (data) {
					histogram(data.hist);
					scatter(data.scatter);
					setCorrelationLabel(data.CC);
				}
			});
		}
		// on page load
		datePickerElement.val('08-2020'); // show Oct 2020 by default
		datePickerElementEnd.val('09-2020'); // show Oct 2020 by default
		//var refId = getRefId(datePickerElement.val());
		//var refIdStart = getRefId(datePickerElement.val());
		var refIdEnd = getRefIdStartEnd(datePickerElement.val(),datePickerElementEnd.val());

		//paintWithRefId(refId);
		paintWithRefId(refIdEnd);

		$('#gobutton').click(function () {
			var refId = getRefId(datePickerElement.val());
			if (refId) {
				paintWithRefId(refId);
			} else {
				alert("Please choose a month");
			}
		});
		$('#gobuttonEnd').click(function () {
			//var refId = getRefId(datePickerElementEnd.val());
			var refId = getRefIdStartEnd(datePickerElement.val(),datePickerElementEnd.val());
			if (refId) {
				paintWithRefId(refId);
			} else {
				alert("Please choose a month");
			}
		});
	</script>
</body>
</html>
