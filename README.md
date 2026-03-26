package com.hs.aiagent

import android.Manifest
import android.content.Intent
import android.content.pm.PackageManager
import android.os.Bundle
import android.speech.RecognitionListener
import android.speech.RecognizerIntent
import android.speech.SpeechRecognizer
import androidx.appcompat.app.AppCompatActivity
import androidx.core.app.ActivityCompat
import androidx.core.content.ContextCompat
import android.widget.Button
import android.widget.TextView
import android.widget.Toast
import java.util.*

class MainActivity : AppCompatActivity() {
    private lateinit var speechRecognizer: SpeechRecognizer
    private lateinit var statusText: TextView
    private val RECORD_REQUEST_CODE = 101

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        statusText = findViewById(R.id.statusText)
        val listenButton: Button = findViewById(R.id.listenButton)

        // Initialize speech recognizer
        speechRecognizer = SpeechRecognizer.createSpeechRecognizer(this)

        listenButton.setOnClickListener {
            if (checkPermissions()) {
                startListening()
            } else {
                requestPermissions()
            }
        }

        // Hindi language support
        speechRecognizer.setRecognitionListener(HindiRecognitionListener())
    }

    private fun checkPermissions(): Boolean {
        return ContextCompat.checkSelfPermission(this, Manifest.permission.RECORD_AUDIO) == PackageManager.PERMISSION_GRANTED
    }

    private fun requestPermissions() {
        ActivityCompat.requestPermissions(this, arrayOf(Manifest.permission.RECORD_AUDIO), RECORD_REQUEST_CODE)
    }

    private fun startListening() {
        val intent = Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH).apply {
            putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL, RecognizerIntent.LANGUAGE_MODEL_FREE_FORM)
            putExtra(RecognizerIntent.EXTRA_LANGUAGE, "hi-IN") // Hindi
            putExtra(RecognizerIntent.EXTRA_PROMPT, "हेलो HS, बोलो...")
            putExtra(RecognizerIntent.EXTRA_MAX_RESULTS, 1)
        }
        speechRecognizer.startListening(intent)
        statusText.text = "सुन रहा हूँ... बोलो!"
    }

    inner class HindiRecognitionListener : RecognitionListener {
        override fun onResults(results: Bundle?) {
            val matches = results?.getStringArrayList(SpeechRecognizer.RESULTS_RECOGNITION)
            matches?.firstOrNull()?.let { command ->
                processHindiCommand(command.lowercase(Locale.getDefault()))
                statusText.text = "आदेश: $command"
            }
            startListening() // Continuous listening
        }

        override fun onError(error: Int) {
            statusText.text = "त्रुटि, फिर से बोलो"
            startListening()
        }

        override fun onReadyForSpeech(params: Bundle?) {}
        override fun onBeginningOfSpeech() {}
        override fun onRmsChanged(rmsdB: Float) {}
        override fun onBufferReceived(buffer: ByteArray?) {}
        override fun onEndOfSpeech() {}
        override fun onPartialResults(partialResults: Bundle?) {}
        override fun onEvent(eventType: Int, params: Bundle?) {}
    }

    private fun processHindiCommand(command: String) {
        when {
            command.contains("कॉल") -> {
                // Open dialer (add phone number logic)
                val intent = Intent(Intent.ACTION_DIAL)
                startActivity(intent)
                Toast.makeText(this, "कॉल डायलर खोल दिया", Toast.LENGTH_SHORT).show()
            }
            command.contains("म्यूजिक") || command.contains("गाना") -> {
                val intent = Intent(Intent.ACTION_VIEW).apply {
                    setDataAndType(android.net.Uri.parse("content://media/external/audio/media"), "audio/*")
                }
                startActivity(intent)
                Toast.makeText(this, "म्यूजिक प्लेयर खोल दिया", Toast.LENGTH_SHORT).show()
            }
            command.contains("लाइट") -> {
                // Control flashlight (add CAMERA permission)
                Toast.makeText(this, "लाइट ऑन/ऑफ (Accessibility via service)", Toast.LENGTH_SHORT).show()
                // Call HSService for advanced control
            }
            command.contains("बंद") -> finish()
            else -> Toast.makeText(this, "समझ नहीं आया: $command", Toast.LENGTH_SHORT).show()
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        speechRecognizer.destroy()
    }
}
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"
    android:padding="20dp">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="🗣️ HS - हिंदी वॉइस असिस्टेंट"
        android:textSize="24sp"
        android:textStyle="bold"
        android:layout_marginBottom="50dp" />

    <TextView
        android:id="@+id/statusText"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="तैयार! 'सुनो' दबाओ"
        android:textSize="18sp"
        android:gravity="center"
        android:layout_marginBottom="30dp" />

    <Button
        android:id="@+id/listenButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="🎤 सुनो HS"
        android:textSize="20sp" />

</LinearLayout>
package com.hs.aiagent

import android.accessibilityservice.AccessibilityService
import android.view.accessibility.AccessibilityEvent
import android.widget.Toast

class HSService : AccessibilityService() {
    override fun onAccessibilityEvent(event: AccessibilityEvent?) {
        // Listen for system events and control phone
        // Example: Simulate button clicks, volume, etc.
    }

    override fun onInterrupt() {}

    override fun onServiceConnected() {
        Toast.makeText(this, "HS सक्रिय! अब फोन कंट्रोल संभव", Toast.LENGTH_LONG).show()
    }
}
<service
    android:name=".HSService"
    android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">
    <intent-filter>
        <action android:name="android.accessibilityservice.AccessibilityService" />
    </intent-filter>
    <meta-data
        android:name="android.accessibilityservice"
        android:resource="@xml/accessibility_service_config" />
</service>
<?xml version="1.0" encoding="utf-8"?>
<accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
    android:accessibilityEventTypes="typeAllMask"
    android:accessibilityFeedbackType="feedbackGeneric"
    android:notificationTimeout="100"
    android:canRetrieveWindowContent="true"
    android:settingsActivity="com.hs.aiagent.MainActivity"
    android:description="@string/accessibility_service_description" />
