# Standar Operating Procedure 

## Content

- [Chaining Methods](#chaining-methods)
- [Methods should do just one thing](#methods-should-do-just-one-thing)

### **Chaining Methods**

Object-oriented programming technique that allows you to call multiple methods on a single object instance in a single, sequential statement.

Bad:

```php
    public function __invoke(Request $request, string $permohonan_id)
    {
        $this->user = $this->getRealUser();
        $aksesPermohonan = $this->getAksesPermohonanUnitKerja($this->user, $permohonan_id);
        if (empty($aksesPermohonan)) return $this->unAuthorized();

        $this->application = $this->tblPermohonanService->findOneBy(['id' => $permohonan_id]);

        $idKpknl = $this->user->getCurrentProfile()->getUnitKerja()->getId();



        $this->penetapanStatusService->insertPenetapanStatus($permohonan_id, false, null, 'kirim', $this->user->getUsername());

        $penetapan = $this->tblPenetapanService->getByPermohonan($permohonan_id);
        if ($penetapan == null) {
            throw new NotFoundException('penetapan');
        }

        return $this->tblPenetapanService->postKirimKepala($this->user, $permohonan_id);
    }
```

Good:

```php
    public function __invoke(Request $request, string $permohonan_id): Response | View
    {
        $this
            //where u first initialized variable with null 
            ->init()
            ->setData()
            ->checkRequirement()
            ->process()

        return $this->success('msg');
    }
```

### **Methods should do just one thing**

A function should do just one thing and do it well.

Bad:

```php
    private function checkRequirement(): self
    {
        if ($this->getAksesPermohonanUnitKerja($this->user, $this->applicationId) === []) {
            $this->unauthorizedException();
        }

        $this->permohonan = $this->tblPermohonanService->get($this->applicationId);

        if (null === $this->permohonan) {
            $this->resourceNotFoundException();
        }

        if(ApplicationStatusEnum::PENETAPAN_JADWAL_LELANG != $this->permohonan->getStatus()->getNama()) {
            $this->badRequestException();
        }

        if ($this->form->get('jumlahPengumuman')->getData() == JumlahPengumumanPenetapanEnum::SATU_KALI) {

            if ($this->form->get('tanggalTerbit')->getData() > $this->form->get('tanggalSelesai')->getData()) {
                $this->badRequestException('Tanggal terbit tidak boleh lebih dari tanggal selesai');
            }

        } else {
            $announcements = $this->form->get('pengumumans')->getData();

            if ($announcements[1]->getTanggalTerbit() < $announcements[0]->getTanggalTerbit()) {
                $this->badRequestException('Tanggal terbit kedua harus lebih dari tanggal terbit pertama');
            }
        }

        return $this;
    }
```

Good:

```php
    private function checkRequirement(): self
    {
        $this
            ->checkUserAccess()
            ->checkApplication()
            ->checkStartEndDate();

        return $this
    }

    private function checkUserAccess(): self
    {
        if ($this->getAksesPermohonanUnitKerja($this->user, $this->applicationId) === []) {
            $this->unauthorizedException();
        }

        return $this;
    }

    private function checkApplication(): self
    {
        $this->permohonan = $this->tblPermohonanService->get($this->applicationId);

        if (null === $this->permohonan) {
            $this->resourceNotFoundException();
        }

        if(ApplicationStatusEnum::PENETAPAN_JADWAL_LELANG != $this->permohonan->getStatus()->getNama()) {
            $this->badRequestException();
        }

        return $this;
    }

    private function checkStartEndDate(): self
    {

        if ($this->form->get('jumlahPengumuman')->getData() == JumlahPengumumanPenetapanEnum::SATU_KALI) {

            if ($this->form->get('tanggalTerbit')->getData() > $this->form->get('tanggalSelesai')->getData()) {
                $this->badRequestException('Tanggal terbit tidak boleh lebih dari tanggal selesai');
            }

        } else {
            $announcements = $this->form->get('pengumumans')->getData();

            if ($announcements[1]->getTanggalTerbit() < $announcements[0]->getTanggalTerbit()) {
                $this->badRequestException('Tanggal terbit kedua harus lebih dari tanggal terbit pertama');
            }

        }

        return $this;
    }
```


