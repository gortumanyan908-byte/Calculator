package com.example.myapplication

import android.os.Bundle
import android.text.format.DateFormat
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Close
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.unit.dp
import java.util.*

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            MaterialTheme {
                CalculatorApp()
            }
        }
    }
}

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun CalculatorApp() {
    var showHistory by remember { mutableStateOf(false) } // Показывать историю?
    var history by remember { mutableStateOf(emptyList<Triple<String, String, String>>()) }

    // Основная страница калькулятора
    CalculatorScreen(
        history = history,
        onShowHistory = { showHistory = true }, // Показать историю
        onDismissHistory = { showHistory = false }, // Закрыть историю
        onAddToHistory = { expression, result ->
            // Получаем текущее время и форматируем его
            val timeStamp = DateFormat.format("yyyy-MM-dd HH:mm:ss", Date(System.currentTimeMillis())).toString()
            history = history + Triple(expression, result, timeStamp)
        },
        onClearHistory = { history = emptyList() } // Очистка истории
    )

    // Диалог с историей вычислений
    if (showHistory) {
        AlertDialog(
            onDismissRequest = { showHistory = false },
            icon = { Icon(Icons.Filled.Close, contentDescription = "Close") },
            title = { Text("История вычислений") },
            text = {
                // Список истории
                LazyColumn(modifier = Modifier.height(400.dp)) {
                    items(history) { entry ->
                        Row(
                            modifier = Modifier
                                .fillMaxWidth()
                                .padding(vertical = 4.dp),
                            horizontalArrangement = Arrangement.SpaceBetween
                        ) {
                            Column {
                                Text("${entry.first} = ${entry.second}", style = MaterialTheme.typography.bodyMedium)
                                Text(entry.third, style = MaterialTheme.typography.labelSmall)
                            }
                        }
                    }
                }
            },
            dismissButton = { // Очистить историю
                TextButton(onClick = {
                    history = emptyList()
                    showHistory = false
                }) {
                    Text("Очистить историю")
                }
            },
            confirmButton = {
                TextButton(onClick = { showHistory = false }) {
                    Text("Закрыть")
                }
            }
        )
    }
}

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun CalculatorScreen(
    history: List<Triple<String, String, String>>, // Теперь храним тройку: выражение, результат, время
    onShowHistory: () -> Unit, // Показать историю
    onDismissHistory: () -> Unit, // Закрыть историю
    onAddToHistory: (String, String) -> Unit, // Добавить в историю
    onClearHistory: () -> Unit // Очистка истории
) {
    var expression by remember { mutableStateOf("") }

    // Функция для вычисления результата выражения
    fun evaluate(expr: String): String {
        try {
            val tokens = mutableListOf<String>()
            var numBuf = ""

            for (char in expr) {
                when (char) {
                    '+', '-', '*', '/' -> {
                        if (numBuf.isNotEmpty()) {
                            tokens.add(numBuf)
                            numBuf = ""
                        }
                        tokens.add(char.toString())
                    }
                    else -> numBuf += char
                }
            }
            if (numBuf.isNotEmpty()) tokens.add(numBuf)

            var result = tokens[0].toDouble()
            var idx = 1
            while (idx < tokens.size) {
                val op = tokens[idx++]
                val number = tokens[idx++].toDouble()

                when (op) {
                    "+" -> result += number
                    "-" -> result -= number
                    "*" -> result *= number
                    "/" -> {
                        if(number != 0.0) result /= number
                        else return "Ошибка: деление на ноль"
                    }
                }
            }
            return result.toString()
        } catch (e: Exception) {
            return "Ошибка"
        }
    }

    Surface(color = MaterialTheme.colorScheme.surface) {
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(horizontal = 16.dp, vertical = 16.dp)
        ) {
            // Текущее выражение
            Text(
                text = if (expression.isEmpty()) "0" else expression,
                style = MaterialTheme.typography.headlineMedium,
                textAlign = TextAlign.End,
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(8.dp)
            )

            // Кнопочная сетка
            val buttonRows = listOf(
                listOf("C", "⌫", "/", "*"),
                listOf("7", "8", "9", "-"),
                listOf("4", "5", "6", "+"),
                listOf("1", "2", "3", "="),
                listOf("0", ".", "История", "") // Новая кнопка "История"
            )

            for (row in buttonRows) {
                Row(
                    modifier = Modifier.fillMaxWidth(),
                    horizontalArrangement = Arrangement.spacedBy(8.dp)
                ) {
                    for ((index, label) in row.withIndex()) {
                        if (label.isNotEmpty()) {
                            Button(
                                onClick = {
                                    when (label) {
                                        "C" -> {
                                            expression = ""
                                            onClearHistory() // Очистка истории
                                        }
                                        "⌫" -> if (expression.isNotEmpty()) expression = expression.dropLast(1)
                                        "=" -> {
                                            val result = evaluate(expression)
                                            if (result != "Ошибка") {
                                                onAddToHistory.invoke(expression, result)
                                            }
                                            expression = result
                                        }
                                        "История" -> { // Показать историю
                                            onShowHistory()
                                        }
                                        else -> {
                                            if (label == "." && '.' in expression) return@Button
                                            expression += label
                                        }
                                    }
                                },
                                colors = ButtonDefaults.buttonColors(
                                    containerColor = Color.Gray,
                                    contentColor = Color.White
                                ),
                                shape = MaterialTheme.shapes.small,
                                modifier = Modifier
                                    .weight(1f)
                                    .height(60.dp)
                            ) {
                                Text(text = label)
                            }
                        } else {
                            Spacer(modifier = Modifier.weight(1f))
                        }
                    }
                }
            }
        }
    }
}

// Предварительный просмотр (убрали @Preview, так как он не использовался)
@Composable
fun CalculatorPreview() {
    MaterialTheme {
        CalculatorApp()
    }
}
