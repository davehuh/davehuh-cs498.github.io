<!DOCTYPE html>
<html>
  <head>
	<meta charset="utf-8">
	<script src="https://d3js.org/d3.v5.min.js"></script>

	<h2> Return on investment on post secondary education in the U.S. (1995 - 2013)</h2>
	<h4> All earnings and costs are fixed to 2017 US dollars</h4>
  </head>


<style> /* set the CSS */
div.tooltip {
	position: absolute;
	text-align: left;
	width: 200px;
	height: 28px;
	padding: 2px;
	font: 12px sans-serif;
	background: lightsteelblue;
	border: 0px;
	border_radius: 10px;
	pointer-events: none;
}

</style>

<body> 
	<!--Select button--> 
	<label for="educationButton">Education Attainment: <br/></label>
	<select id="educationButton"></select> 
	
	<!--Color scale--> 
	<script src="https://d3js.org/d3-scale-chromatic.v1.min.js"></script>


	<div id="earningdegviz"></div>
	<div id="description"></br>Tuitions have been steadily arising throughout the years. Same with earnings. Between the costs of education and their returns on investment, this visualization illustrates a message about which levels of education tend to yield more earnings. <br/><br/> Return on investment or ROI is calculated by accounting for three variables: Median earnings of selected education group, Aggregate median earnings across education groups, and tuition cost of selected education group. The formula in a way normalizes benefits and costs from higher and lower educations by comparing to aggregate median earnings. Higher the ROI the better.</div>


<script>

	var margin = {top: 10, right: 30, bottom: 30, left: 50},
		width = 700 - margin.left - margin.right,
		height = 450 - margin.top - margin.bottom;
	
	var div = d3.select("#earningdegviz").append("div")
			.attr("class", "tooltip")
			.style("opacity", 0);

	var svg = d3.select("#earningdegviz")
		.append("svg")
			.attr("width", width + margin.left + margin.right)
			.attr("height", height + margin.top + margin.bottom)
		.append("g")
			.attr("transform",
				"translate(" + margin.left + "," + margin.top + ")");

<!--Read data-->
  d3.csv("https://raw.githubusercontent.com/davehuh/davehuh-cs498.github.io/master/data/data.csv",
	  function(d) {
		return {
			Year : d3.timeParse("%Y")(d.Year),
			Median_Earning : +d["Median Earning"],
			Tuition : +d.Tuition,
			Education_Attainment : d["Education Attainment"],
			ROI : +d.ROI,
			All : +d.All
		};

	  }).then(function(data) {

		  //List of groups
		  var allEducationGroups = ["<Select Option>", "Some college, no degree", "Associates", "Bachelors", "Masters or higher degree"]

		  // Responsive description module
		  var info = [
			  {key: "<Select Option>", 
				  value: 
				  "Tuitions have been steadily arising throughout the years. Same with earnings. Between the costs of education and their returns on investment, this visualization illustrates a message about which levels of education tend to yield more earnings. <br/><br/> Return on investment or ROI is calculated by accounting for three variables: Median earnings of selected education group, Aggregate median earnings across education groups, and tuition cost of selected education group. The formula in a way normalizes benefits and costs from higher and lower educations by comparing to aggregate median earnings. Higher the ROI the better."
			  },
			  {key: "Some college, no degree", 
				  value: 
				  "People who attend college but never earn a degree do not experience increasing in earnings from going to school. Returns tend to be close to zero or on the negative side."
			  },
			  {key: "Associates", 
				  value: 
				  "Two year programs or Associates degrees are affordable options to gain post secondary education in US. Because of its low tuition cost, the return on investment tends to be on the positive."
			  },
			  {key: "Bachelors", 
				  value: 
				  "Undergraduate tuition has much higher starting cost compared to the Associates degree. And the costs for Bachelors degrees have been rising steadily throughout the period, doubled from 1995 and 2013."
			  },
			  {key: "Masters or higher degree", 
				  value: 
				  "People with masters degree or higher earn more than those with undergraduate degrees. Compared to undergraduate tuition, graduate tuition has maintained similar costs. The difference is that the entire ROI line is higher than the line for Bachelors."
			  }
		  ]

		  //Options for select button
		  d3.select("#educationButton")
		    .selectAll('myOptions')
		       .data(allEducationGroups)
		    .enter()
		       .append('option')
		    .text(function (d) { return d; })
		    .attr("value", function(d) {return d;})


		  // Color scale one for each education group
		  var myColor = d3.scaleOrdinal()
		    .domain(allEducationGroups)
		    .range(d3.schemeSet2);


		  var x = d3.scaleTime()
		    .domain(d3.extent(data, function(d) { return d.Year; }))
		    .range([0, width]);
		  svg.append("g")
		    .attr("transform", "translate(0," + height +")")
		    .call(d3.axisBottom(x));


		  var y = d3.scaleLinear()
		    .domain([d3.min(data, function(d) { return +d.ROI;  }), d3.max(data, function(d) { return +d.ROI;  })])
		    .range([height,0]);
		  svg.append("g")
		    .call(d3.axisLeft(y));

		  var initialData = data.filter(function(d, i)
			  {
				  if (d["Education_Attainment"] == "All")
				  {
					return d;
				  }
			  }

		  )

		  var line = svg
		    .append("path")
		    .datum(initialData)
		    .attr("fill", "none")
	            .attr("stroke", function(d) {return myColor("All")})
		    .attr("stroke-width", 1.5)
		    .attr("d", d3.line()
		      .x(function(d) { return x(d.Year); })
		      .y(function(d) { return y(d.ROI); })
		        )


		  var formatYear = d3.timeFormat("%Y")
		  var formatCurrency = d3.format(",")

		  // Dots used for tooltip
		  var dot = svg
		    .selectAll("dot")
		        .data(initialData)
	            .enter().append("circle")
		        .attr("r", 4)
			  .attr("cx", function(d) { return x(d.Year); })
			  .attr("cy", function(d) { return y(d.ROI); })
		          .on("mouseover", function(d)
				  {
					  div.transition()
					     .duration(200)
					     .style("opacity", .9);
					  div.html("Year: " + formatYear(d.Year) + "<br/>" +
						  "ROI: " + d.ROI + "<br/>" +	
						  "Earning across education: " + formatCurrency(d.All) + "<br/>" + 
						  "Median Earning: " + formatCurrency(d.Median_Earning) + "<br/>" +
						  "Tuition: " + formatCurrency(d.Tuition
					  ))
					  .style("left", (d3.event.pageX) + "px")
					  .style("top", (d3.event.pageY - 28) + "px");
				  })
		    .on("mouseout", function(d)
			    {
				    div.transition()
				       .duration(500)
				       .style("opacity", 0);
			    }

		    )


		  // An update function for the chart
		  function update(selectedGroup) {

			  // New data with the selection
			  var filterEducationData = data.filter(function(d, i)
				  {
					  if (d["Education_Attainment"] == selectedGroup)
					  {
						  return d;
					  }
				  }
			  )

			  // update line
			  line
			      .datum(filterEducationData)
			      .transition()
			      .duration(1000)
			      .attr("d", d3.line()
     		                 .x(function(d) { return x(d.Year); })
     		                 .y(function(d) { return y(d.ROI); })
			  )
				  .attr("stroke", function(d) { return myColor(selectedGroup) })

                          // update dots
			  dot
			      .data(filterEducationData)
			      .transition()
			      .duration(1000)
		              .attr("r", 4)
			      .attr("cx", function(d) { return x(d.Year); })
			      .attr("cy", function(d) { return y(d.ROI); })

                          var i;
			  var description;
			  for (i = 0; i < info.length; i++)
			  {
			     if (selectedGroup == info[i].key)
			     {
				  description = info[i].value;
		   	     }
			  }

			  document.getElementById("description").innerHTML = description
		  }

		  // button selection and update
		  d3.select("#educationButton").on("change", function(d)
			  {
				  var selectedOption = d3.select(this).property("value")
				  update(selectedOption)
			  }
		  )

<!--formula legend-->
		  svg.append("text")
		          .attr("text-anchor", "end")
			  .attr("x", width )
		  	  .attr("y", height - 20)
		          .text("ROI = (Median Earning - Earning across education) / Tuition")
		          .style("font-size", "10px")
		          .attr("alignment-baseline", "middle")

		  // x-axis label
		  svg.append("text")
		          .attr("class", "x label")
		          .attr("text-anchor", "end")
			  .attr("x", width / 2)
		  	  .attr("y", height + 25)
		          .text("Year")
		          .style("font-size", "12px")
		          .attr("alignment-baseline", "middle")

		  // y-axis label
		  svg.append("text")
		          .attr("class", "y label")
		          .attr("text-anchor", "end")
			  .attr("x", -width/3)
		  	  .attr("y", height/2 - 250)
		          .text("ROI")
		          .style("font-size", "12px")
		          .attr("transform", "rotate(-90)")
		          .attr("alignment-baseline", "middle")
	  });
  </script>
</body>
</html>
