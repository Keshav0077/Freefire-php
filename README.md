# Freefire-php
<?php
$botToken = "8072308599:AAGkiDcH94KKFb4R8waggB9JkbIe7cZLffY";
$botAPI = "https://api.telegram.org/bot$botToken/";
$authorizedGroups = ['-1002287360773'];

function sendMessage($chatId, $text, $messageId = null, $parseMode = "HTML")
{
    global $botAPI;
    $data = [
        'chat_id' => $chatId,
        'text' => $text,
        'parse_mode' => $parseMode,
    ];
    if ($messageId) {
        $data['reply_to_message_id'] = $messageId;
    }
    $response = file_get_contents($botAPI . "sendMessage?" . http_build_query($data));
    file_put_contents('telegram_debug_log.txt', "SendMessage Response: $response\n", FILE_APPEND);
    return $response;
}

function editMessage($chatId, $messageId, $text, $parseMode = "HTML")
{
    global $botAPI;
    $data = [
        'chat_id' => $chatId,
        'message_id' => $messageId,
        'text' => $text,
        'parse_mode' => $parseMode,
    ];
    $response = file_get_contents($botAPI . "editMessageText?" . http_build_query($data));
    file_put_contents('telegram_debug_log.txt', "EditMessage Response: $response\n", FILE_APPEND);
    return $response;
}

$update = json_decode(file_get_contents('php://input'), true);

if (isset($update['message'])) {
    $message = $update['message'];
    $chatId = $message['chat']['id'];
    $text = $message['text'] ?? '';
    $messageId = $message['message_id'];
    $groupId = $message['chat']['id'];

    if (!in_array($groupId, $authorizedGroups)) {
        sendMessage($chatId, "⚠️ This group is not authorized to use the bot.\n\n🤯 MESSAGE ME HERE TO ADD BOT TO YOUR GROUP👉 @Searching7_filebot", $messageId);
        exit;
    }

    if (preg_match('/^\/like (\w+) (\d+)$/', $text, $matches)) {
        $region = strtoupper($matches[1]);
        $uid = $matches[2];

        $processingMessage = sendMessage($chatId, "⏳ Please wait, processing your request...", $messageId);
        $processingMessageId = json_decode($processingMessage, true)['result']['message_id'] ?? null;

        if (!$processingMessageId) {
            file_put_contents('error_log.txt', "Failed to send processing message.\n", FILE_APPEND);
            exit;
        }

        $apiUrl = "https://vstech.serv00.net/freeapi.php?uid={uid}&region={region}&key=narayan";
        $apiResponse = file_get_contents($apiUrl);
        file_put_contents('api_debug_log.txt', "API Response: $apiResponse\n", FILE_APPEND);

        if ($apiResponse === false) {
            editMessage($chatId, $processingMessageId, "❌ Failed to contact the API. Please try again later.");
            exit;
        }

        $apiResponse = json_decode($apiResponse, true);

        if (isset($apiResponse['LikesGivenByAPI']) && $apiResponse['LikesGivenByAPI'] == 0) {
            $responseText = "UID <b>$uid</b> in region <b>$region</b> has already received Max Likes for Today. Please try a different UID.";
        } elseif (isset($apiResponse['LikesGivenByAPI'])) {
            $responseText = "✨ <b>LIKE SENDED SUCCESS</b> ✨\n\n";
            $responseText .= "👤 <b>NAME:</b> " . htmlspecialchars($apiResponse['PlayerNickname'], ENT_QUOTES, 'UTF-8') . "\n";
            $responseText .= "🌐 <b>REGION:</b> $region 🇮🇳\n";
            $responseText .= "👍 <b>LIKES SENDED:</b> " . $apiResponse['LikesGivenByAPI'] . "\n";
            $responseText .= "♨️ <b>LIKE BEFORE COMMAND:</b> " . $apiResponse['LikesbeforeCommand'] . "\n";
            $responseText .= "🌠 <b>LIKE AFTER COMMAND:</b> " . $apiResponse['LikesafterCommand'] . "\n";
            $responseText .= "☘️ <b>API RUN:</b> SUCCESS\n\n";
            $responseText .= "🤖 <b>ALSO JOIN:</b> @Moviespoint077bot\n";
            $responseText .= "👾 <b>BOT WAS MADE BY:</b> @Keshavraj_77";
        } else {
            $responseText = "❌ Unexpected API response. Please try again later.";
        }

        editMessage($chatId, $processingMessageId, $responseText);
    } else {
        exit;
    }
}
?>
