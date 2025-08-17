# New-star
<!DOCTYPE html>
<html lang="mn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mongol Casino</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="container">
        <!-- Хэрэглэгчийн мэдээлэл -->
        <div class="user-panel">
            <div class="balance-box">
                <span>Баланс: </span>
                <span id="user-balance">0</span>₮
            </div>
            <button id="deposit-btn">Цэнэглэх</button>
        </div>

        <!-- Цэнэглэх форм -->
        <div id="deposit-modal" class="modal">
            <div class="modal-content">
                <span class="close">&times;</span>
                <h2>Данс цэнэглэх</h2>
                <form id="deposit-form">
                    <input type="text" id="phone-number" placeholder="Утасны дугаар" required>
                    <input type="number" id="amount" placeholder="Дүн" required>
                    <input type="text" id="transaction-code" placeholder="Гүйлгээний код" required>
                    <button type="submit">Илгээх</button>
                </form>
            </div>
        </div>

        <!-- Тоглоомын интерфейс -->
        <div class="games-container">
            <!-- Тоглоомын жагсаалт энд -->
        </div>
    </div>

    <script src="script.js"></script>
</body>
</html>
.modal {
    display: none;
    position: fixed;
    z-index: 1;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0,0,0,0.7);
}

.modal-content {
    background: #1e1e1e;
    margin: 15% auto;
    padding: 20px;
    width: 80%;
    max-width: 500px;
    border-radius: 8px;
}

#deposit-form input {
    width: 100%;
    padding: 10px;
    margin: 10px 0;
    border-radius: 4px;
    border: 1px solid #333;
    background: #252525;
    color: white;
}
// Хэрэглэгчийн мэдээлэл
let user = {
    balance: 0,
    phone: ''
};

// Цэнэглэх форм илгээх
document.getElementById('deposit-form').addEventListener('submit', async function(e) {
    e.preventDefault();
    
    const phone = document.getElementById('phone-number').value;
    const amount = parseFloat(document.getElementById('amount').value);
    const txCode = document.getElementById('transaction-code').value;

    // Сервер рүү илгээх
    const response = await fetch('deposit.php', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            phone: phone,
            amount: amount,
            transaction_code: txCode
        })
    });

    const result = await response.json();
    
    if (result.success) {
        user.balance += amount;
        updateBalance();
        alert('Амжилттай цэнэглэгдлээ!');
        closeModal();
    } else {
        alert('Алдаа: ' + result.message);
    }
});

// Баланс шинэчлэх
function updateBalance() {
    document.getElementById('user-balance').textContent = user.balance.toLocaleString();
}
<?php
header('Content-Type: application/json');

// MySQL холболт
$servername = "localhost";
$username = "root";
$password = "";
$dbname = "casino_db";

$conn = new mysqli($servername, $username, $password, $dbname);

// Гүйлгээний мэдээлэл
$data = json_decode(file_get_contents('php://input'), true);
$phone = $data['phone'];
$amount = $data['amount'];
$txCode = $data['transaction_code'];

// 1. Гүйлгээ баталгаажуулах
$isValidTransaction = verifyTransaction($txCode, $amount); // Бодит системд банкны API холбоно

if ($isValidTransaction) {
    // 2. Хэрэглэгчийн баланс шинэчлэх
    $sql = "UPDATE users SET balance = balance + ? WHERE phone = ?";
    $stmt = $conn->prepare($sql);
    $stmt->bind_param("ds", $amount, $phone);
    $stmt->execute();
    
    // 3. Гүйлгээний түүх хадгалах
    $txSql = "INSERT INTO transactions (phone, amount, tx_code) VALUES (?, ?, ?)";
    $txStmt = $conn->prepare($txSql);
    $txStmt->bind_param("sds", $phone, $amount, $txCode);
    $txStmt->execute();
    
    echo json_encode(['success' => true]);
} else {
    echo json_encode(['success' => false, 'message' => 'Гүйлгээ баталгаажаагүй']);
}

function verifyTransaction($txCode, $amount) {
    // Энд банкны API холбож баталгаажуулах
    return true; // Demo-д зориулж үргэлж true буцаана
}
?>
