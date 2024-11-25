<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Invoice with Web Data Store</title>
    <style>
        /* Print Styles */
        @media print {
            body * {
                visibility: hidden;
            }
            #printableInvoice, #printableInvoice * {
                visibility: visible;
            }
            #printableInvoice {
                position: absolute;
                left: 0;
                top: 0;
                width: 100%;
            }
        }

        /* Non-Print Styles */
        body {
            background-color: #6A0DAD;
            font-family: Arial, sans-serif;
        }

        input {
            background-color: #b5d8e1;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            margin-bottom: 20px;
        }

        th, td {
            padding: 8px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }

        th {
            background-color: #00a2ff;
        }

        button {
            padding: 8px 16px;
            background-color: #c60000;
            color: white;
            border: none;
            cursor: pointer;
            border-radius: 5px;
            transition: all 0.3s ease-in-out;
        }

        button:hover {
            background-color: #a00000;
        }
    </style>
</head>
<body>
    <h1 align="center"><font face="Segoe Print"><b>INVOICE</b></font></h1>
    <form id="billingForm" onsubmit="createInvoice(event)">
        <hr>
        <table>
            <tr>        
                <td><label for="sellerName">Seller Name:</label></td>
                <td><input type="text" id="sellerName" required><br><br></td>
            </tr>
            <tr>
                <td><label for="buyerName">Buyer Name:</label></td>
                <td><input type="text" id="buyerName" required><br><br></td>
            </tr>
            <tr>
                <td><label for="buyerMobile">Buyer Mobile Number:</label></td>
                <td><input type="tel" id="buyerMobile" pattern="[0-9]{10}" required><br><br></td>
            </tr>
        </table>

        <table id="invoiceTable">
            <thead>
                <tr>
                    <th>Item Name</th>
                    <th>Quantity</th>
                    <th>Price</th>
                    <th>Total</th>
                    <th>Action</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td><input type="text" name="itemName[]" required></td>
                    <td><input type="number" name="quantity[]" min="1" required onchange="calculateItemTotal(this)"></td>
                    <td><input type="number" name="price[]" min="0" step="0.01" required onchange="calculateItemTotal(this)"></td>
                    <td><input type="text" name="total[]" readonly></td>
                    <td><button type="button" onclick="removeItem(this)">Remove</button></td>
                </tr>
            </tbody>
        </table>

        <button type="button" onclick="addItem()">Add Item</button><br><br>
        <label for="grandTotal">Grand Total:</label>
        <input type="text" id="grandTotal" readonly><br><br>
        <button type="submit">Create and Save Invoice</button>
    </form>

    <h2>Saved Invoices</h2>
    <div id="savedInvoices"></div>

    <!-- Printable Invoice Layout -->
    <div id="printableInvoice" style="display: none;">
        <h1>INVOICE</h1>
        <p><b>Seller Name:</b> <span id="printSellerName"></span></p>
        <p><b>Buyer Name:</b> <span id="printBuyerName"></span></p>
        <p><b>Buyer Mobile:</b> <span id="printBuyerMobile"></span></p>
        <p><b>Date:</b> <span id="printDate"></span></p>
        <table>
            <thead>
                <tr>
                    <th>Item Name</th>
                    <th>Quantity</th>
                    <th>Price</th>
                    <th>Total</th>
                </tr>
            </thead>
            <tbody id="printItems"></tbody>
        </table>
        <p><b>Grand Total: Rs.<span id="printGrandTotal"></span></b></p>
    </div>

    <script>
        function addItem() {
            const tableBody = document.querySelector('#invoiceTable tbody');
            const row = document.createElement('tr');
            row.innerHTML = `
                <td><input type="text" name="itemName[]" required></td>
                <td><input type="number" name="quantity[]" min="1" required onchange="calculateItemTotal(this)"></td>
                <td><input type="number" name="price[]" min="0" step="0.01" required onchange="calculateItemTotal(this)"></td>
                <td><input type="text" name="total[]" readonly></td>
                <td><button type="button" onclick="removeItem(this)">Remove</button></td>
            `;
            tableBody.appendChild(row);
        }

        function calculateItemTotal(input) {
            const row = input.closest('tr');
            const quantity = parseFloat(row.querySelector('[name="quantity[]"]').value) || 0;
            const price = parseFloat(row.querySelector('[name="price[]"]').value) || 0;
            row.querySelector('[name="total[]"]').value = (quantity * price).toFixed(2);
            calculateGrandTotal();
        }

        function calculateGrandTotal() {
            const totals = document.querySelectorAll('[name="total[]"]');
            let grandTotal = 0;
            totals.forEach(total => grandTotal += parseFloat(total.value) || 0);
            document.getElementById('grandTotal').value = grandTotal.toFixed(2);
        }

        function removeItem(button) {
            button.closest('tr').remove();
            calculateGrandTotal();
        }

        function createInvoice(event) {
            event.preventDefault();

            const sellerName = document.getElementById('sellerName').value;
            const buyerName = document.getElementById('buyerName').value;
            const buyerMobile = document.getElementById('buyerMobile').value;

            const items = Array.from(document.querySelectorAll('#invoiceTable tbody tr')).map(row => ({
                name: row.querySelector('[name="itemName[]"]').value,
                quantity: row.querySelector('[name="quantity[]"]').value,
                price: row.querySelector('[name="price[]"]').value,
                total: row.querySelector('[name="total[]"]').value,
            }));

            const grandTotal = document.getElementById('grandTotal').value;

            const invoice = {
                sellerName,
                buyerName,
                buyerMobile,
                items,
                grandTotal,
                date: new Date().toLocaleString(),
            };

            saveInvoice(invoice);
            displayInvoices();
            alert('Invoice saved successfully!');
            document.getElementById('billingForm').reset();
            document.getElementById('grandTotal').value = '';
        }

        function saveInvoice(invoice) {
            const invoices = JSON.parse(localStorage.getItem('invoices')) || [];
            invoices.push(invoice);
            localStorage.setItem('invoices', JSON.stringify(invoices));
        }

        function displayInvoices() {
            const invoices = JSON.parse(localStorage.getItem('invoices')) || [];
            const savedInvoices = document.getElementById('savedInvoices');
            savedInvoices.innerHTML = invoices.map((invoice, index) => `
                <div>
                    <p><b>Seller:</b> ${invoice.sellerName}, <b>Buyer:</b> ${invoice.buyerName}, <b>Date:</b> ${invoice.date}</p>
                    <p><b>Grand Total:</b> Rs.${invoice.grandTotal}</p>
                    <button onclick="deleteInvoice(${index})">Delete</button>
                    <button onclick="printInvoice(${index})">Print</button>
                </div>
            `).join('');
        }

        function deleteInvoice(index) {
            const invoices = JSON.parse(localStorage.getItem('invoices')) || [];
            invoices.splice(index, 1);
            localStorage.setItem('invoices', JSON.stringify(invoices));
            displayInvoices();
        }

        function printInvoice(index) {
            const invoices = JSON.parse(localStorage.getItem('invoices')) || [];
            const invoice = invoices[index];

            document.getElementById('printSellerName').textContent = invoice.sellerName;
            document.getElementById('printBuyerName').textContent = invoice.buyerName;
            document.getElementById('printBuyerMobile').textContent = invoice.buyerMobile;
            document.getElementById('printDate').textContent = invoice.date;

            const printItems = document.getElementById('printItems');
            printItems.innerHTML = invoice.items.map(item => `
                <tr>
                    <td>${item.name}</td>
                    <td>${item.quantity}</td>
                    <td>${item.price}</td>
                    <td>${item.total}</td>
                </tr>
            `).join('');

            document.getElementById('printGrandTotal').textContent = invoice.grandTotal;

            const printableInvoice = document.getElementById('printableInvoice');
            printableInvoice.style.display = 'block';
            window.print();
            printableInvoice.style.display = 'none';
        }

        window.onload = displayInvoices;
    </script>
</body>
</html>
 
