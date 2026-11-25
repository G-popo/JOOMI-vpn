
package com.joomi.vpn;

import android.content.Intent;
import android.net.VpnService;
import android.os.Bundle;
import android.webkit.JavascriptInterface;
import android.widget.Toast;
import com.getcapacitor.BridgeActivity;

public class MainActivity extends BridgeActivity {

    private static final int VPN_REQUEST_CODE = 1001;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // Pont JavaScript -> Android
        this.bridge.getWebView().addJavascriptInterface(new Object(){

            @JavascriptInterface
            public void activeMonForfait(String code) {
                runOnUiThread(() -> {
                    Toast.makeText(MainActivity.this, "Activation: " + code, Toast.LENGTH_SHORT).show();
                    startJoomiVpn(code);
                });
            }

            @JavascriptInterface
            public void stopVpn() {
                runOnUiThread(() -> stopJoomiVpn());
            }

        }, "Android");
    }

    private void startJoomiVpn(String code) {
        Intent intent = VpnService.prepare(MainActivity.this);
        if (intent != null) {
            startActivityForResult(intent, VPN_REQUEST_CODE);
        } else {
            // Permission déjà accordée
            onActivityResult(VPN_REQUEST_CODE, RESULT_OK, null);
        }
        // Sauvegarde le code pour le onActivityResult
        getIntent().putExtra("pending_code", code);
    }

    private void stopJoomiVpn() {
        Intent intent = new Intent(this, JoomiVpnService.class);
        stopService(intent);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == VPN_REQUEST_CODE && resultCode == RESULT_OK) {
            String code = getIntent().getStringExtra("pending_code");
            Intent serviceIntent = new Intent(this, JoomiVpnService.class);
            serviceIntent.putExtra("code", code != null ? code : "UNKNOWN");
            startService(serviceIntent);
        }
    }
}
