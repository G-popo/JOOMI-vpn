
package com.joomi.vpn;

import android.content.Intent;
import android.net.VpnService;
import android.os.ParcelFileDescriptor;
import android.util.Log;
import java.io.IOException;
import java.nio.channels.FileChannel;

public class JoomiVpnService extends VpnService {
    private ParcelFileDescriptor vpnInterface = null;
    private static final String TAG = "JoomiVpn";

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        String code = intent != null ? intent.getStringExtra("code") : "unknown";
        Log.d(TAG, "Démarrage VPN avec code: " + code);

        // Demande de permission - Android va afficher la clé automatiquement quand on établit l'interface
        try {
            Builder builder = new Builder()
                    .addAddress("10.0.0.2", 32)
                    .addRoute("0.0.0.0", 0)
                    .addDnsServer("8.8.8.8")
                    .addDnsServer("1.1.1.1")
                    .setSession("JOOMI VPN")
                    .setMtu(1500);

            // C'est cette ligne qui fait apparaître la clé 🔑 à côté de 4G
            vpnInterface = builder.establish();

            if (vpnInterface != null) {
                Log.d(TAG, "VPN Interface établie - icône clé visible !");
                // Ici tu brancheras Xray/V2Ray pour faire passer le vrai internet
                // Pour l'instant on garde l'interface ouverte pour montrer la clé
            }

        } catch (Exception e) {
            Log.e(TAG, "Erreur VPN", e);
        }

        return START_STICKY;
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        try {
            if (vpnInterface != null) {
                vpnInterface.close();
                vpnInterface = null;
                Log.d(TAG, "VPN arrêté - clé disparue");
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void onRevoke() {
        stopSelf();
        super.onRevoke();
    }
}
