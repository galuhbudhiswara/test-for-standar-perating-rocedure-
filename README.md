# Standar Operating Procedure 

## Contents

- [Chaining Methods](#chaining-methods)
- [Validate Every Payload](#validate-every-payload)
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
            //where u first initialized variable with null value
            ->init()
            //where u assign variable with the actual value
            ->setData()
            //handle if data null or another case validate 
            ->checkRequirement()
            //the actual endpoint process 
            ->process()

        //return endpoint response 
        return $this->success('msg');
    }
```


[🔝 Back to contents](#contents)


### **Validate Every Payload**

Move validation from controllers to Request classes.

Bad:

```php
    public function __invoke(Request $request, string $permohonan_id)
    {
        $payload = $request->request->all();
        $userWhatsappNumber = $payload['nomor'];
    }
```

Good:

```php
    public function __invoke(Request $request)
    {
        $form = $this->formFactory->submitRequest(TblUserWhatsappType::class, $request);
        if (!$form->isValid()) {
            return $this->view($form, Response::HTTP_BAD_REQUEST);
        }
    }
```

```php
    final class TblUserWhatsappType extends AbstractType
    {
        public function buildForm(FormBuilderInterface $builder, array $options)
        {
            $builder->add('nomor', null, [
                'required' => true,
                'label' => 'sas.form.field.tblorganisasi.nomor',
                'attr' => [
                    'placeholder' => '+62812xxxxxxxx',
                    'pattern' => '[0-9]{7,15}'
                ],
                'constraints' => [
                    new NotBlank([
                        'message' => 'Nomor telepon tidak boleh kosong'
                    ]),
                    new NotNull([
                        'message' => 'Nomor telepon harus diisi'
                    ]),
                    new Regex([
                        'pattern' => '/^[0-9]{7,15}$/', 
                        'message' => 'Nomor telepon harus menggunakan format internasional dan hanya mengandung angka'
                    ]),
                    new Callback([
                        'callback' => [$this, 'validateIndonesianNumber']
                    ])
                ]
            ]);
        }

        public function validateIndonesianNumber($value, ExecutionContextInterface $context)
        {
            if (strpos($value, '62') === 0) {
                if (strlen($value) > 2 && $value[2] !== '8') {
                    $context->buildViolation('Nomor telepon Indonesia harus diawali dengan +628')
                        ->addViolation();
                }
            }
        }

        public function configureOptions(OptionsResolver $resolver)
        {
            $resolver->setDefaults([
                'translation_domain' => 'forms',
            ]);
        }
    }
```


[🔝 Back to contents](#contents)


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


[🔝 Back to contents](#contents)


### **Entity Should Have Setter and Getter**

It is widely considered a best practice for entities in object-oriented programming (OOP) to have getters and setters for their private fields. This approach, known as encapsulation.

Bad:

```php
    namespace KejawenLab\Application\Entity;

    use KejawenLab\Application\Repository\TblUserBankRepository;
    use Symfony\Component\Serializer\Annotation\Groups;
    use Doctrine\ORM\Mapping as ORM;
    use Ramsey\Uuid\Doctrine\UuidGenerator;
    use Ramsey\Uuid\UuidInterface;
    use Gedmo\Blameable\Traits\BlameableEntity;
    use Gedmo\Mapping\Annotation as Gedmo;
    use Gedmo\SoftDeleteable\Traits\SoftDeleteableEntity;
    use Gedmo\Timestampable\Traits\TimestampableEntity;
    use KejawenLab\Application\Controller\v1\Traits\SecurityTrait;
    use KejawenLab\Application\TblUserWhatsapp\Model\TblUserWhatsappInterface;

    #[ORM\Table(name: 'tbl_user_nomor_whatsapp', schema: 'registrasi')]
    #[ORM\Entity(repositoryClass: TblUserBankRepository::class)]
    #[Gedmo\SoftDeleteable(fieldName: "deletedAt")]

    class TblUserWhatsapp implements TblUserWhatsappInterface
    {
        #[ORM\Id]
        #[ORM\Column(type: "uuid", unique: true)]
        #[ORM\GeneratedValue(strategy: "CUSTOM")]
        #[ORM\CustomIdGenerator(class: UuidGenerator::class)]
        #[Groups(groups: ['read'])]
        protected UuidInterface|string $id;

        #[ORM\ManyToOne]
        #[ORM\JoinColumn(nullable: false)]
        private ?TblUser $user = null;

        #[ORM\Column]
        #[Groups(groups: ['read'])]
        private ?bool $active = null;

        #[ORM\Column(length: 50)]
        #[Groups(groups: ['read', 'entitas_reg'])]
        private ?string $nomorTelepon = null;
    }
```

Good:

```php
    namespace KejawenLab\Application\Entity;

    use KejawenLab\Application\Repository\TblUserBankRepository;
    use Symfony\Component\Serializer\Annotation\Groups;
    use Doctrine\ORM\Mapping as ORM;
    use Ramsey\Uuid\Doctrine\UuidGenerator;
    use Ramsey\Uuid\UuidInterface;
    use Gedmo\Blameable\Traits\BlameableEntity;
    use Gedmo\Mapping\Annotation as Gedmo;
    use Gedmo\SoftDeleteable\Traits\SoftDeleteableEntity;
    use Gedmo\Timestampable\Traits\TimestampableEntity;
    use KejawenLab\Application\Controller\v1\Traits\SecurityTrait;
    use KejawenLab\Application\TblUserWhatsapp\Model\TblUserWhatsappInterface;

    #[ORM\Table(name: 'tbl_user_nomor_whatsapp', schema: 'registrasi')]
    #[ORM\Entity(repositoryClass: TblUserBankRepository::class)]
    #[Gedmo\SoftDeleteable(fieldName: "deletedAt")]

    class TblUserWhatsapp implements TblUserWhatsappInterface
    {
        use BlameableEntity;
        use SoftDeleteableEntity;
        use TimestampableEntity;

        #[ORM\Id]
        #[ORM\Column(type: "uuid", unique: true)]
        #[ORM\GeneratedValue(strategy: "CUSTOM")]
        #[ORM\CustomIdGenerator(class: UuidGenerator::class)]
        #[Groups(groups: ['read'])]
        protected UuidInterface|string $id;

        #[ORM\ManyToOne]
        #[ORM\JoinColumn(nullable: false)]
        private ?TblUser $user = null;

        #[ORM\Column]
        #[Groups(groups: ['read'])]
        private ?bool $active = null;

        #[ORM\Column(length: 50)]
        #[Groups(groups: ['read', 'entitas_reg'])]
        private ?string $nomorTelepon = null;


        public function getId(): ?string
        {
            return $this->id;
        }

        public function getUser(): ?TblUser
        {
            return $this->user;
        }

        public function setUser(?TblUser $user): self
        {
            $this->user = $user;

            return $this;
        }

        public function getNomorTelepon(): ?string
        {
            return $this->nomorTelepon;
        }

        public function setNomorTelepon(?string $nomorTelepon): self
        {
            $this->nomorTelepon = $nomorTelepon;

            return $this;
        }

        public function isActive(): ?bool
        {
            return $this->active;
        }
        
        public function setActive(bool $active): self
        {
            $this->active = $active;

            return $this;
        }

        public function getNullOrString(): ?string
        {
            return $this->getNomorTelepon();
            
        }

    }

```



