---
title: Arsitektur
---

Aplikasi didesain menggunakan DDD (_domain-driven design_) dan _onion
architecture_. _Onion architecture_ adalah arsitektur yang memisahkan antara
business logic dengan teknologi yang digunakan. Dengan _onion architecture_,
aplikasi akan lebih mudah diuji, dikembangkan, dan dipelihara.

Aplikasi kita dibagi menjadi beberapa layer, dimulai dari layer terdalam:

* Layer **domain**. Namespace `App\Domain`. Terdiri dari entity, value object
  dan objek-objek pendukung, seperti collection, domain event, exception, dan
  lain-lain.

* Layer **domain service**. Namespace `App\DomainService`. Terdiri dari
  repository, event listener, dan service lain yang hanya memiliki dependency ke
  layer domain.

* Layer **application service**. Namespace `App\ApplicationService`. Terdiri
  dari application-level event listener, messenger handler, dan service
  semacamnya.

* Layer-layer luar, terdiri dari:
  
  * Layer **infrastructure**. Namespace `App\Infrastructure`. Terdiri dari
    implementasi dari repository, event listener, dan service lain yang
    berinteraksi dengan teknologi eksternal.

  * Layer **front end**. Berkaitan dengan interaksi aplikasi dengan pengguna.
    Terdiri dari:

    * Layer **web front end**. Namespace `App\WebFrontEnd`. Terdiri dari
      web controller, form, template, dan service lain yang berhubungan dengan
      web.
    
    * Layer **API front end**. Namespace `App\ApiFrontEnd`. Terdiri dari
      API resource, API state, konfigurasi mapper, dan sebagainya.
    
    * Layer **console front end**. Namespace `App\Command`. Terdiri dari
      console command, dan service lain yang berhubungan.

Aturan dasarnya adalah:

* Layer luar boleh mengakses layer dalam, tapi layer dalam tidak boleh
  mengakses layer di atasnya.
* Layer dalam boleh membuat interface. Dan layer luar yang membuat
  implementasinya. Konsep ini dinamakan _dependency inversion principle_.