# go-foxpro-dbf

[![GoDoc](https://godoc.org/github.com/golang/gddo?status.svg)](http://godoc.org/github.com/SebastiaanKlippert/go-foxpro-dbf)
[![Build Status](https://travis-ci.org/SebastiaanKlippert/go-foxpro-dbf.svg?branch=master)](https://travis-ci.org/SebastiaanKlippert/go-foxpro-dbf)


Golang package for reading FoxPro DBF/FPT files.

This package provides a reader for reading FoxPro database files.
At this moment it is only tested for Alaska XBase++ DBF/FPT files in FoxPro format.
These files have file flag 0x30 (or 0x31 if autoincrement fields are present).

Since these files are almost always used on Windows platforms the default encoding is
from Windows-1250 to UTF8 but a universal encoder will be provided for other codepages.

# Features 

There are several similar packages but they are not suited for our use case, this package will try to implement:
* Support for FPT (memo) files
* Full support for Windows-1250 encoding to UTF8
* Filereaders for scanning files (instead of reading the entire file to memory)

The focus is on performance while also trying to keep the code readable and easy to use.

# Example

```go

func Test() error {
	//Open file
	dbf, err := dbf.OpenFile("TEST.DBF")
	if err != nil {
		return err
	}
	defer dbf.Close()

	//Loop through all records using recordpointer in DBF struct
	//Reads the complete record
	for i := uint32(0); i < dbf.NumRecords(); i++ {

		//This reads the complete record
		record, err := dbf.Record()
		if err != nil {
			return err
		}
		dbf.Skip(1)

		//skip deleted records
		if record.Deleted {
			continue
		}
		//get field by position
		field1, err := record.Field(0)
		if err != nil {
			return err
		}
		//get field by name
		field2, err := record.Field(dbf.FieldPos("NAAM"))
		if err != nil {
			return err
		}

		fmt.Println(field1, field2)
	}

	//Read only the third field of records 2, 30 and 50
	recnumbers := []uint32{2, 50, 300}
	for _, rec := range recnumbers {
		err := dbf.GoTo(rec)
		if err != nil {
			return err
		}
		deleted, err := dbf.Deleted()
		if err != nil {
			return err
		}
		if !deleted {
			field3, err := dbf.Field(3)
			if err != nil {
				return err
			}
			fmt.Println(field3)
		}
	}

	return nil
}
```