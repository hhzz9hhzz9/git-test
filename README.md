# git-test
package main

import (
	"crypto/rand"
	"errors"
	"fmt"
	"math/big"
)

type CharacterType string

const (
	specialArray                 = "!\"#$%&'()*+,-./\\:;<=>?@[]^`{_|}~"
	letterRange                  = 26
	numberRange                  = 10
	special        CharacterType = "SPECIAL"
	lower          CharacterType = "LOWER"
	upper          CharacterType = "UPPER"
	number         CharacterType = "NUMBER"
	lowerBegin                   = 97
	upperBegin                   = 65
	numberBegin                  = 48
	minVariousType               = 4
)

func getRandomInt(s string) byte {
	i, err := rand.Int(rand.Reader, big.NewInt(int64(len(s))))
	if err != nil {
		panic("生成密码失败: " + err.Error())
	}
	return s[i.Int64()]
}

func main() {
	s,_ := generateRandomPassword(10)
	fmt.Println(s)
}

func generateRandomPassword(length int) (string, error) {
	content := make([]uint8, length, length)
	pwCharsIndex := make([]int, length, length)
	for i, _ := range pwCharsIndex {
		pwCharsIndex[i] = i
	}
	takeTypes := []CharacterType{lower, upper, number, special}
	fixedTypes := []CharacterType{lower, upper, number, special}
	typeCount := 0
	for len(pwCharsIndex) > 0 {
		index, err := rand.Int(rand.Reader, big.NewInt(int64(len(pwCharsIndex))))
		if err != nil {
			return "", err
		}
		pwIndex := pwCharsIndex[index.Int64()]
		pwCharsIndex = append(pwCharsIndex[:index.Int64()], pwCharsIndex[index.Int64()+1:]...)
		var character byte
		if typeCount < minVariousType {
			takeIndex, err := rand.Int(rand.Reader, big.NewInt(int64(len(takeTypes))))
			if err != nil {
				return "", err
			}
			if character, err = generateCharacter(takeTypes[takeIndex.Int64()]); err != nil {
				return "", err
			}
			takeTypes = append(takeTypes[:takeIndex.Int64()], takeTypes[takeIndex.Int64()+1:]...)
			typeCount++
		}else {
			fixedIndex, err := rand.Int(rand.Reader, big.NewInt(int64(len(fixedTypes))))
			if err != nil {
				return "", err
			}
			if character, err = generateCharacter(fixedTypes[fixedIndex.Int64()]); err != nil {
				return "", err
			}
			content[pwIndex] = character
		}
	}
	return string(content), nil
}

func sliceRemove(s *[]interface{}, index int) interface{} {
	tmp := (*s)[index]
	*s = append((*s)[:index], (*s)[index+1:]...)
	return tmp
}

func generateCharacter(stringType CharacterType) (uint8, error) {
	switch stringType {
	case lower: // 随机小写
		i, err := rand.Int(rand.Reader, big.NewInt(int64(letterRange)))
		if err != nil {
			return 0, err
		}
		return uint8(i.Int64() + lowerBegin), nil
	case upper: // 随机大写
		i, err := rand.Int(rand.Reader, big.NewInt(int64(letterRange)))
		if err != nil {
			return 0, err
		}
		return uint8(i.Int64() + upperBegin), nil
	case number: // 随机数字
		i, err := rand.Int(rand.Reader, big.NewInt(int64(numberRange)))
		if err != nil {
			return 0, err
		}
		return uint8(i.Int64() + numberBegin), nil
	case special: // 随机字符
		i, err := rand.Int(rand.Reader, big.NewInt(int64(len(specialArray))))
		if err != nil {
			return 0, err
		}
		return specialArray[i.Int64()], nil
	}
	return 0, errors.New("byte zero")
}

