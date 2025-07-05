Saran pak andri: Collateral disable untuk Go Live PM 1: Matiin Penjagaan statusInventory di function searchNoSbg, Ketika pelunasan, antara ketika create atau cetak kwitansi, lakuin proses kayak collateral out .

Collateral out tambah search ic, pilih statusInventory,Note,sama Other pengennya gausah diinput, jadi default aja statusInventory=Out, Note=Other, Other=Default.

Perlu bikin managemen commit per branch/feature. misal lagi ngubah buffer cabang, ada branch sendiri feature/buffer cabang, nanti di dev baru dimerge, tapi perlu ada metode bisa bedain mana yang masih di dev, mana yang bisa naek prod.

Feature->On Dev->On Dev QA Done->Prod
Flow nya: feature di merge ke On Dev, ketika feature specific lolos QA, merge ke Done QA, merge lagi ke Prod



customer mau bisa nambahin nomor telp ke 2 dan bisa nampilin juga. renewal bisa ubah berat

Gajadi -- dengan ada penambahan idValuerRenewal yang ngubah nya. Ketika ada renewal lagi, idValuerRenewal yang diubah

API untuk limit pengubahan berat renewal, bisa pake param isActive, dengan field limit decimal pengubahan, dari database.



