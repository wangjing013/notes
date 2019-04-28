# MySQL



## 建表语法



```mysql
DROP TABLE IF EXISTS `users`;

CREATE TABLE IF NOT EXISTS `users` (
`id` INTEGER NOT NULL auto_increment ,
`firstName` VARCHAR(20),
`lastName` VARCHAR(20), 
`age` INTEGER,
`createdAt` DATETIME NOT NULL, 
`updatedAt` DATETIME NOT NULL,
 PRIMARY KEY (`id`)
) ENGINE=InnoDB;
```

