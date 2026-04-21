/**
 * ARCHITECT & ABCABCCOL - WW-VIEWER CORE ENGINE
 * Özellikler: Veri Akışı, Dosya Bağlama, Saniyelik Otomatik Kayıt
 */

interface WWFile {
  name: string;
  content: string;
  type: 'html' | 'css' | 'js' | 'json';
}

class WWViewerEngine {
  private files: WWFile[] = [];
  private saveInterval: any = null;

  constructor(initialFiles: WWFile[]) {
    this.files = initialFiles;
    this.startInfiniteLoop(); // Kısır döngüyü başlat
  }

  // 1. VERİ AKIŞI: Tüm dosyaları birleştirip aktif hale getirir
  private compileProject() {
    const html = this.files.find(f => f.name.endsWith('.html'))?.content || '';
    const css = this.files.filter(f => f.name.endsWith('.css')).map(f => f.content).join('\n');
    const js = this.files.filter(f => f.name.endsWith('.js')).map(f => f.content).join('\n');
    const userData = this.files.find(f => f.name === 'user_info.json')?.content || '{}';

    return `
      <!DOCTYPE html>
      <html>
        <head>
          <style>${css}</style>
          <script>
            // Kullanıcı Bilgileri Enjeksiyonu
            window.WW_USER_DATA = ${userData};
          </script>
        </head>
        <body>
          ${html}
          <script>${js}<\/script>
        </body>
      </html>
    `;
  }

  // 2. KISIR DÖNGÜ & SANİYELİK KAYIT: Her saniye durumu kontrol eder
  private startInfiniteLoop() {
    console.log("WW-Viewer: Kısır döngü ve Veri Akışı aktif.");

    this.saveInterval = setInterval(() => {
      this.autoSave();
    }, 1000); // 1000ms = 1 Saniye
  }

  // 3. KULLANICI BİLGİLERİNİ KAYDETME: Her saniye veritabanına yazar
  private autoSave() {
    try {
      const currentStatus = {
        timestamp: new Date().toISOString(),
        files: this.files,
        systemStorage: "81.3GB_FREE_SYNC" // Sistem bilgisini de korur
      };

      // Tarayıcı hafızasına (veya sunucuya) saniyelik kayıt
      localStorage.setItem('WW_PERSISTENCE_LAYER', JSON.stringify(currentStatus));
      
      // Mimari log (Opsiyonel)
      // console.log("WW-Viewer: Veri Akışı Senkronize Edildi."); 
    } catch (err) {
      console.error("Kritik Kayıt Hatası:", err);
    }
  }

  // 4. WW'NİN AÇILMASI: Görüntüleyiciyi başlatır
  public launch() {
    const finalBundle = this.compileProject();
    const blob = new Blob([finalBundle], { type: 'text/html' });
    return URL.createObjectURL(blob);
  }
}