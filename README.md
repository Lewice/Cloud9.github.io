<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Menu Calculator and Form Submission</title>
  <script src="https://code.jquery.com/jquery-3.4.1.js" integrity="sha256-WpOohJOqMqqyKL9FccASB9O0KwACQJpFTUBLTYOVvVU=" crossorigin="anonymous"></script>
  <style>
    body, h2, form, label, p, button, select, input {
      font-size: 10;
      margin-right: 10px; /* Add a margin for spacing */
    }

    label {
      display: block; /* Display as inline-block to make items appear beside each other */
      margin-bottom: 5px;
    }

    body, h2, form {
      text-align: center;
    }

    p {
      text-align: center;
      margin: 0; /* Remove default margin for <p> */
    }

    button {
      margin-top: 10px;
    }
	body {
	background-color: grey;
	}
  </style>
  <script>
    function calculateTotals() {
  let total = 0;

  // Calculate total from selected items
  const menuItems = document.querySelectorAll('.menu-item:checked');
  menuItems.forEach(item => {
    const price = parseFloat(item.dataset.price);
    const quantity = parseInt(item.nextElementSibling.value);

    if (!isNaN(price) && !isNaN(quantity) && quantity > 0) {
      // Exclude "Mystery Box" from discounts
      if (item.classList.contains('exclude-discount')) {
        total += price * quantity;
      } else {
        total += price * quantity * (1 - ($("#discount").val() / 100));
      }
    }
  });

  // Change commission rate from 5% to 10%
  const commission = total * 0.50;

  document.getElementById('total').innerText = total.toFixed(2);
  document.getElementById('commission').innerText = commission.toFixed(2);
}

   function SubForm() {
  // Check if the employee name, customer state ID, and customer name are provided
  const employeeName = $("#employeeName").val();
  const customerStateID = $("#customerStateID").val();
  const customerName = $("#customerName").val();

  if (employeeName.trim() === "") {
    alert("Employee Name is required!");
    return;
  }

  // Get selected menu items and quantities
  const orderedItems = [];
  const menuItems = document.querySelectorAll('.menu-item:checked');
  menuItems.forEach(item => {
    const itemName = item.parentNode.textContent.trim();
    const price = parseFloat(item.dataset.price);
    const quantity = parseInt(item.nextElementSibling.value);

    if (!isNaN(price) && !isNaN(quantity) && quantity > 0) {
      orderedItems.push({
        name: itemName,
        price: price,
        quantity: quantity
      });
    }
  });

  // Calculate total and commission
  const total = parseFloat($("#total").text());
  const commission = parseFloat($("#commission").text());
  const discount = parseFloat($("#discount").val());

  // Prepare data for API submission
  const formData = {
    "Employee Name": employeeName,
    "Total": total.toFixed(2),
    "Commission": commission.toFixed(2),
    "Customer State ID": customerStateID,
    "Customer Name": customerName,
    "Items Ordered": orderedItems.map(item => `${item.quantity}x ${item.name}`).join('\n')
  };


  // Form Submission Logic for Spreadsheet
  $.ajax({
    url: "https://api.apispreadsheets.com/data/jivCDcIqUL1gO3yP/",
    type: "post",
    data: formData,
    headers: {
      accessKey: "eadc12d3473490193b58b016fdefbe6a",
      secretKey: "90025e0efdf69f727ca58f86f1e83bae",
      "Content-Type": "application/x-www-form-urlencoded",
    },
    success: function () {
      alert("Form Data Submitted to Spreadsheet and Discord :)");
      resetForm();
    },
    error: function () {
      alert("There was an error :(");
    }
  });

  // Prepare data for Discord webhook
  const discordData = {
    username: "Cloud9's Recipts",
    content: `New order submitted by ${employeeName}`,
    embeds: [{
      title: "Order Details",
      fields: [
        { name: "Employee Name", value: employeeName, inline: true },
        { name: "Customer State ID", value: customerStateID, inline: true },
        { name: "Customer Name", value: customerName, inline: true },
        { name: "Total", value: `$${total.toFixed(2)}`, inline: true },
        { name: "Commission", value: `$${commission.toFixed(2)}`, inline: true },
        { name: "Discount Applied", value: `${discount}%`, inline: true },
        { name: "Items Ordered", value: orderedItems.map(item => `${item.quantity}x ${item.name}`).join('\n') }
      ],
      color: 0x00ff00 // You can customize the color
    }]
  };

  // Form Submission Logic for Discord webhook
  $.ajax({
    url: "https://discord.com/api/webhooks/1177304010034778122/LWkVYw9WCXkOXGvGCdb1-0p3TShKxxxovQExUWuY36Lihp1abXbr92msFGEHWcCbYaom",
    type: "post",
    contentType: "application/json",
    data: JSON.stringify(discordData),
    success: function () {
      // Do nothing specific for Discord success
    },
    error: function () {
      console.error("Error sending data to Discord :(");
    }
  });

  // Reset checkboxes and quantity inputs
  $('.menu-item').prop('checked', false);
  $('.quantity').val(1);

  // Reset totals
  document.getElementById('total').innerText = '';
  document.getElementById('commission').innerText = '';
  // Reset discount dropdown to default
  $("#discount").val("0");
}
  </script>
</head>
<body>

  <h2>Cloud9 Drift</h2>

  <form id="menuForm">
  <h3>Rentals</h3>
    <label>
      <input type="checkbox" class="menu-item" data-price="2000"> 30 Min Rental w/ 1 Free repair - $2,000
      <input type="number" class="quantity" value="1" min="1">
    </label>
    <label>
      <input type="checkbox" class="menu-item" data-price="4000"> 1 HR Rental w/ 1 free repair - $4,000
      <input type="number" class="quantity" value="1" min="1">
    </label>
    <label>
      <input type="checkbox" class="menu-item" data-price="500"> Additional Repairs - $500
      <input type="number" class="quantity" value="1" min="1">
    </label>
	
	<h3>Lessons</h3>
    <label>
      <input type="checkbox" class="menu-item" data-price="300"> 30Min Lesson - $5,000
      <input type="number" class="quantity" value="1" min="1">
    </label>
    <label>
      <input type="checkbox" class="menu-item" data-price="300"> 1Hr Lesson - $10,000
      <input type="number" class="quantity" value="1" min="1">
    </label>
    
	
	<h3>Certifications</h3>
    <label>
      <input type="checkbox" class="menu-item" data-price="10000"> Certification 1 - $10,000
      <input type="number" class="quantity" value="1" min="1">
    </label>
	<label>
      <input type="checkbox" class="menu-item" data-price="10000"> Certification 2 - $10,000
      <input type="number" class="quantity" value="1" min="1">
    </label>
	<label>
      <input type="checkbox" class="menu-item" data-price="10000"> Certification 3 - $10,000
      <input type="number" class="quantity" value="1" min="1">
    </label>
	
	<h3>License</h3>
    <label>
      <input type="checkbox" class="menu-item" data-price="25000"> License - $25,000
      <input type="number" class="quantity" value="1" min="1">
    </label>
	<div style="margin-bottom: 30px;"></div>
	
	<h3>Customer Information</h3>
	
	<div style="margin-bottom: 15px;"></div>
	
	<label for="customerStateID">Customer State ID:</label>
	<input type="text" id="customerStateID">
	<label for="customerName">Customer Name:</label>
	<input type="text" id="customerName">
    
	<div style="margin-bottom: 30px;"></div>
	
	<label for="discount">Select Discount:</label>
    <select id="discount" onchange="calculateTotals()">
      <option value="0">No Discount</option>
    </select>
	
	<div style="margin-bottom: 30px;"></div>
	
    <label for="employeeName">Employee Name:</label>
    <input type="text" id="employeeName" required>
	
	<div style="margin-bottom: 30px;"></div>
	
    <p>Total: $<span id="total"></span></p>
    <p>Commission (50%): $<span id="commission"></span></p>
	
	<div style="margin-bottom: 30px;"></div>

    <button type="button" onclick="calculateTotals()">Calculate</button>
    <button type="button" onclick="SubForm()">Submit</button>
    <button type="button" onclick="resetForm()">Reset</button>
  </form>

</body>
</html>
