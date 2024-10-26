package main

import (
	"bufio"
	"fmt"
	"math"
	"math/rand"
	"os"
	"sort"
	"strconv"
	"strings"
	"time"
)
func SliceFill(arr *[]int,cords *[][2]int,size int){
	var min3,max3=-100000000,100000000
	var min2,max2=-100,100
	for j:=0;j<size;j++{
		(*arr)[j]=rand.Intn(max3-min3+1) + min3
		(*cords)[j][0]=rand.Intn(max2-min2+1) + min2
		(*cords)[j][1]=rand.Intn(max2-min2+1) + min2
	}
}
func createCordFile(i int,cords [][2]int){
	cordsFileName := "Coordinates" + strconv.Itoa(i+1) + ".txt"
	file,err:=os.Create(cordsFileName)
	if err != nil {
			fmt.Println("Error creating file:", err)
			return
	}
	
	defer file.Close()
		for _, coord := range cords {
		if _, err := fmt.Fprintf(file, "%d %d\n", coord[0], coord[1]); err != nil {
			fmt.Println("Error writing to file")
		}
	}
		
}
func createIntFile(i int,arr []int){
	intMultiplyFileName := "Integers" + strconv.Itoa(i+1) + ".txt"
	file,err:=os.Create(intMultiplyFileName)
	if err != nil {
			fmt.Println("Error creating file:", err)
			return
	}
	
	defer file.Close()
		for _, val := range arr {
		if _, err := fmt.Fprintf(file, "%d\n", val); err != nil {
			fmt.Println("Error writing to file")
		}
	}
}
func display_cordFiles(input int)int{
	fmt.Println("0-Go Back")
	var opt int
	for i:=0;i<input;i++{
		cordsFileName := "Coordinates" + strconv.Itoa(i+1) + ".txt"
		fmt.Printf("%v-%v\n",i+1,cordsFileName)
	}
	fmt.Println("Enter Option number:")
	fmt.Scanln(&opt)
	if opt>=0 && opt<=input{
		return opt
	}else{
		fmt.Println("Invalid Option!")
		return display_cordFiles(input)
	}
}
func display_intFiles(input int)int{
	var opt int
	fmt.Println("0-Go Back")
	for i:=0;i<input;i++{
		intFileName := "Integers" + strconv.Itoa(i+1) + ".txt"
		fmt.Printf("%v-%v\n",i+1,intFileName)
	}
	fmt.Println("Enter Option number:")
	fmt.Scanln(&opt)
	if opt>=0 && opt<=input{
		return opt
	}else{
		fmt.Println("Invalid Option!")
		return display_intFiles(input)
	}
}
func Display_algo() int{
	var opt int
	fmt.Println("0-End Program")
	fmt.Println("1-Integer Multiplication")
	fmt.Println("2-Closest pair of coordinates")
	fmt.Println("Enter Option number:")
	fmt.Scanln(&opt)
	if opt>=0 && opt<=2{
		return opt
	}else{
		fmt.Println("Invalid Option!")
		return Display_algo()
	}
}
func open_read_intFile(num int)[] int{
	intMultiplyFileName := "Integers" + strconv.Itoa(num) + ".txt"
	file, err := os.Open(intMultiplyFileName) 
	if err != nil {
		fmt.Println("Error opening file:", err)
		
	}
	defer file.Close()
	var arr=make([]int,0)
	scanner := bufio.NewScanner(file)
	for i:=0;scanner.Scan();i++ {
		line := scanner.Text()
		number, err := strconv.Atoi(line)
		if err != nil {
			fmt.Println("Error converting string to integer:", err)
			continue
		}
		arr = append(arr, number)
	}
	if err := scanner.Err(); err != nil {
		fmt.Println("Error reading file:", err)
	}
	return arr
}
func open_read_cordsFile(num int)[][]int{
	intCordsFileName := "Coordinates" + strconv.Itoa(num) + ".txt"
	file, err := os.Open(intCordsFileName) 
	if err != nil {
		fmt.Println("Error opening file:", err)
	}
	defer file.Close()
	var cords=make([][]int,0)
	scanner := bufio.NewScanner(file)
	for i:=0;scanner.Scan();i++ {
		line := scanner.Text()
		parts:=strings.Fields(line)
		if len(parts)<2{
			fmt.Println("Line does not contain 2 numbers")
		}
		num1,err1:=strconv.Atoi(parts[0])
		num2,err2:=strconv.Atoi(parts[1])
		if err1 != nil || err2!=nil {
			fmt.Println("Error converting string to integer:", err)
			continue
		}
		temp:=make([]int,2)
		temp[0]=num1;
		temp[1]=num2;
		cords=append(cords,temp)

	}
	if err := scanner.Err(); err != nil {
		fmt.Println("Error reading file:", err)
	}
	return cords
}
func initialize_files(input int){
	var min,max=100,1000
	var i,size int
	rand.Seed(time.Now().UnixNano())
	for i=0;i<input;i++{
		size=rand.Intn(max-min+1) + min
    	arr:=make([]int,size)
		cords := make([][2]int,size)
		SliceFill(&arr,&cords,size)
		createCordFile(i,cords)
		createIntFile(i,arr)
    }
}
func calculate_distance(c1 []int, c2 []int) float32 {
    return float32(math.Sqrt(math.Pow(float64(c2[0]-c1[0]), 2) + math.Pow(float64(c2[1]-c1[1]), 2)))
}
func Closest_cord_pair(cords [][]int) ([]int, []int, float32) {
	n := len(cords)
	if n <= 3 {
		return bruteForce(cords)
	}
	mid := n / 2
	left := cords[:mid]
	right := cords[mid:]
	p1, p2, dist1 := Closest_cord_pair(left)
	p3, p4, dist2 := Closest_cord_pair(right)
	var minDist float32
	var closestPair1, closestPair2 []int
	if dist1 < dist2 {
		minDist = dist1
		closestPair1, closestPair2 = p1, p2
	} else {
		minDist = dist2
		closestPair1, closestPair2 = p3, p4
	}
	strip := make([][]int, 0)
	lineX := cords[mid][0]
	for _, p := range cords {
		if float32(math.Abs(float64(p[0]-lineX))) < minDist {
			strip = append(strip, p)
		}
	}
	sort.Slice(strip, func(i, j int) bool {
		return strip[i][1] < strip[j][1]
	})
	for i := 0; i < len(strip); i++ {
		for j := i + 1; j < len(strip) && float32((strip[j][1]-strip[i][1])) < minDist; j++ {
			distance := calculate_distance(strip[i], strip[j])
			if distance < minDist {
				minDist = distance
				closestPair1, closestPair2 = strip[i], strip[j]
			}
		}
	}

	return closestPair1, closestPair2, minDist
}
func bruteForce(cords [][]int) ([]int, []int, float32) {
	minDist := float32(math.Inf(1))
	var closestPair1, closestPair2 []int
	n := len(cords)
	for i := 0; i < n; i++ {
		for j := i + 1; j < n; j++ {
			distance := calculate_distance(cords[i], cords[j])
			if distance < minDist {
				minDist = distance
				closestPair1, closestPair2 = cords[i], cords[j]
			}
		}
	}
	return closestPair1, closestPair2, minDist
}
func divide_and_conquer(x, y int) int {
	if x < 10 || y < 10 {
		return x * y
	}
	n := int(math.Min(float64(len(fmt.Sprintf("%d", x))), float64(len(fmt.Sprintf("%d", y)))))
	mid := n / 2
	highX := x / int(math.Pow(10, float64(mid)))
	lowX := x % int(math.Pow(10, float64(mid)))
	highY := y / int(math.Pow(10, float64(mid)))
	lowY := y % int(math.Pow(10, float64(mid)))
	z0 := divide_and_conquer(lowX, lowY)
	z1 := divide_and_conquer((lowX + highX), (lowY + highY))
	z2 := divide_and_conquer(highX, highY)
	return z2*int(math.Pow(10, float64(2*mid))) + (z1-z2-z0)*int(math.Pow(10, float64(mid))) + z0
}
func IntMultiplication(arr []int, num int) []int {
	for i, v := range arr {
		arr[i] = divide_and_conquer(v, num)
	}
	return arr
}
func main() {
	input:=10
    var opt2 int
	initialize_files(input)
	for opt:=-1 ;opt!=0;{
		opt=Display_algo()
		if opt==2{
			opt2=display_cordFiles(input)
			if opt2!=0{
				cords:=open_read_cordsFile(opt2)
				sort.Slice(cords, func(i, j int) bool {
      		    return cords[i][0] < cords[j][0]})
				fmt.Println(cords)
				cord1,cord2,distance:=Closest_cord_pair(cords)
				fmt.Printf("The minimum distance is between %v and %v = %v\n",cord1,cord2,distance)
			}
		}else if opt==1{
			opt2=display_intFiles(input)
			if opt2!=0{
				arr:=open_read_intFile(opt2)
				fmt.Println(arr)
				var num int
				fmt.Println("Enter number to multiply:")
				fmt.Scanln(&num)
				arr=IntMultiplication(arr,num)
				fmt.Println(arr)
			}
		}
	}
}
