```
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "main"
  }
}


resource "aws_subnet" "sub1" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.0.0/22"

  tags = {
    Name = "sub1"
  }
}

resource "aws_subnet" "sub2" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.4.0/22"

  tags = {
    Name = "sub2"
  }
}

resource "aws_internet_gateway" "i-gw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "internet-getway"
  }
}

resource "aws_eip" "elastic" {
  domain = "vpc"
}

resource "aws_nat_gateway" "nat-getway" {
  allocation_id = aws_eip.elastic.id
  subnet_id     = aws_subnet.sub1.id


  tags = {
    name = "nat-getway"

  }
}

resource "aws_route_table" "rout-igw" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.i-gw.id
  }

  tags = {
    name = "Public-rt"

  }
}

resource "aws_route_table" "rout-nat" {
  vpc_id = aws_vpc.main.id


  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_nat_gateway.nat-getway.id
  }

  tags = {
    name = "private-rt"

  }
}

resource "aws_route_table_association" "tr-as" {
  subnet_id      = aws_subnet.sub1.id
  route_table_id = aws_route_table.rout-igw.id
}



resource "aws_route_table_association" "rt-nat" {
  subnet_id      = aws_subnet.sub2.id
  route_table_id = aws_route_table.rout-nat.id
}
```
