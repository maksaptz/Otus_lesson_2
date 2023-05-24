# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
#Name_VM
  :otuslinux => {
	#тип ОС
        :box_name => "centos/7",
	#Добавляем диски
	:disks => {
		:sata1 => {
			:dfile => './sata1.vdi',
			:size => 250,
			:port => 1
		},
		:sata2 => {
                        :dfile => './sata2.vdi',
                        :size => 250, # Megabytes
			:port => 2
		},
                :sata3 => {
                        :dfile => './sata3.vdi',
                        :size => 250,
                        :port => 3
                },
                :sata4 => {
                        :dfile => './sata4.vdi',
                        :size => 250, # Megabytes
                        :port => 4
                },
		:sata5 => {
                        :dfile => './sata5.vdi',
                        :size => 250, # Megabytes
                        :port => 5
                }

	}

		
  },
}

#Начинает цикл, подставляет вместо Vagrant.configure(2) > config
Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name] #Выбираем тип ОС
          box.vm.host_name = boxname.to_s #Имя системы
          box.vm.provider :virtualbox do |vb| #Выбираем virtual box в качестве провайдера виртуализации
            	  vb.customize ["modifyvm", :id, "--memory", "1024"] #Назначаем 1024 мб оперативной памяти
                  needsController = false
		  boxconfig[:disks].each do |dname, dconf| #Циклы которые настраивают диски
			  unless File.exist?(dconf[:dfile])
				vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
                                needsController =  true
                          end

		  end
                  if needsController == true
                     vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                     boxconfig[:disks].each do |dname, dconf|
                         vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                     end
                  end
          end

 	  box.vm.provision "shell", inline: <<-SHELL #Стартовый скрипт
	    yum install -y mdadm #Устанавливает утилиту для настройки Raid
	    mdadm --zero-superblock --force /dev/sd{b,c,d,e,f} #Зануляет суперблоки дисков
            mdadm --create --verbose --force /dev/md0 -l 5 -n 5 /dev/sd{b,c,d,e,f}  #Создает raid5 из 5 дисков
            mkdir /etc/mdadm/ #Создает директорию для файла конфига mdadm
            echo "DEVICE partitions" > /etc/mdadm/mdadm.conf #Создает файл конфигурации mdadm
            mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf #Заполняет файл конфигурации служебной информацией
            parted -s /dev/md0 mklabel gpt #Создает раздел GPT
            parted /dev/md0 mkpart primary ext4 0% 20% #Создаем 5 партиций по 1/5 дискового пространства
            parted /dev/md0 mkpart primary ext4 20% 40%
            parted /dev/md0 mkpart primary ext4 40% 60%
            parted /dev/md0 mkpart primary ext4 60% 80%
            parted /dev/md0 mkpart primary ext4 80% 100%
            for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done #Цикл который создает файловую систему ext4 на дисках
            mkdir -p /raid/part{1,2,3,4,5} #Создаем папку для будущего монтирования
            for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done #Монтируем каждый раздел в соответствующую папку
            echo "#NEW DEVICE" >> /etc/fstab #При помощи цикла настраиваем автомонтирование разделов
            for i in $(seq 1 5); do echo `sudo blkid /dev/md0p$i | awk '{print $2}'` /raid/part$i ext4 defaults 0 0 >> /etc/fstab; done
  	  SHELL
      end
  end
end
