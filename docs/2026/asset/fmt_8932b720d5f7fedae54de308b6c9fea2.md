
```
{ 0xA9, transfer2 },
{ 0xAA, transfer2 },
{ 0xAB, transfer2Mod },
{ 0xAC, transfer2Mod1 },
{ 0xAD, transfer2Mod },
{ 0xAE, transfer2Mod },
{ 0xAF, transfer2Mod },
{ 0xB0, transfer2Mod },
{ 0xB1, transfer2Mod },
{ 0xB2, transfer2Mod },
{ 0xB3, transfer2Mod },
{ 0xB4, transfer2Mod },
{ 0xB5, transfer2Mod },
{ 0xB6, transfer2Mod },
{ 0xB7, transfer2Mod },
{ 0xB8, noTransfer },
{ 0xB9, noTransfer },
{ 0xBA, transfer2Mod1 },
{ 0xBB, transfer2Mod },
{ 0xBC, transfer2Mod },
{ 0xBD, transfer2Mod },
{ 0xBE, transfer2Mod },
{ 0xBF, transfer2Mod },
{ 0xC0, transfer2Mod },
{ 0xC1, transfer2Mod },
{ 0xC2, noTransfer },
{ 0xC3, noTransfer },
{ 0xC4, noTransfer },
{ 0xC5, noTransfer },
{ 0xC6, noTransfer },
{ 0xC7, transfer2Mod },
{ 0xC8, transfer2 },
{ 0xC9, transfer2 },
{ 0xCA, transfer2 },
{ 0xCB, transfer2 },
{ 0xCC, transfer2 },
{ 0xCD, transfer2 },
{ 0xCE, transfer2 },
{ 0xCF, transfer2 },
{ 0xD0, noTransfer },
{ 0xD1, transfer2Mod },
{ 0xD2, transfer2Mod },
{ 0xD3, transfer2Mod },
{ 0xD4, noTransfer },
{ 0xD5, transfer2Mod },
{ 0xD6, noTransfer },
{ 0xD7, noTransfer },
{ 0xD8, transfer2Mod },
{ 0xD9, transfer2Mod },
{ 0xDA, noTransfer },
{ 0xDB, transfer2Mod },
{ 0xDC, transfer2Mod },
{ 0xDD, transfer2Mod },
{ 0xDE, noTransfer },
{ 0xDF, transfer2Mod },
{ 0xE0, noTransfer },
{ 0xE1, transfer2Mod },
{ 0xE2, transfer2Mod },
{ 0xE3, noTransfer },
{ 0xE4, noTransfer },
{ 0xE5, transfer2Mod },
{ 0xE6, noTransfer },
{ 0xE7, noTransfer },
{ 0xE8, transfer2Mod },
{ 0xE9, transfer2Mod },
{ 0xEA, noTransfer },
{ 0xEB, transfer2Mod },
{ 0xEC, transfer2Mod },
{ 0xED, transfer2Mod },
{ 0xEE, noTransfer },
{ 0xEF, transfer2Mod },
{ 0xF0, noTransfer },
{ 0xF1, transfer2Mod },
{ 0xF2, transfer2Mod },
{ 0xF3, transfer2Mod },
{ 0xF4, noTransfer },
{ 0xF5, transfer2Mod },
{ 0xF6, noTransfer },
{ 0xF7, noTransfer },
{ 0xF8, transfer2Mod },
{ 0xF9, transfer2Mod },
{ 0xFA, transfer2Mod },
{ 0xFB, noTransfer },
{ 0xFC, transfer2Mod },
{ 0xFD, transfer2Mod },
{ 0xFE, transfer2Mod },
{ 0xFF, noTransfer },
{ 0x00, lastEntry }
};

PBYTE transferInstruction( PBYTE destination, PBYTE source, PBYTE *jumpAddress,
LONG *extra )
{
X86_16BIT_INSTRUCTION op16 = { 0 };
X86INSTRUCTION* opPtr = { 0 };
*jumpAddress = TARGETLESS_X86INSTRUCTION;
*extra = 0;
op16.operandIs16 = 0;
op16.addressIs16 = 0;
op16.jumpAddress = jumpAddress;
op16.extra = extra;
opPtr = &instructionMap[source[0]];
return opPtr->pXferFunction( &op16, opPtr, destination, source );
}

PBYTE transferData( X86_16BIT_INSTRUCTION* op16Ptr, X86INSTRUCTION* opPtr, PBYTE
destination, PBYTE source )
{
LONG bytes = 0;
LONG fixedBytes = (opPtr->flagMask & ADDRESS_FLAG)
  ? (op16Ptr->addressIs16 ? opPtr->size16 : opPtr->size)
  : (op16Ptr->operandIs16 ? opPtr->size16 : opPtr->size);
bytes = fixedBytes;
if( opPtr->modeOffset > 0 )
{
  BYTE rmMode = source[opPtr->modeOffset];
  BYTE flags = regMemMode[rmMode];
  if( flags & SIB_FLAG )
  {
   if( ( source[opPtr->modeOffset + 1] & 0x07 ) == 0x05 )
   {
    if( ( rmMode & 0xc0 ) == 0x00 )
     bytes += 4;
    else if( ( rmMode & 0xc0 ) == 0x40 )
     bytes += 1;
    else if( ( rmMode & 0xc0 ) == 0x80 )
     bytes += 4;
   }
  }
  bytes += flags & NOTSIB_FLAG;
}
memcpy( destination, source, bytes );
if( opPtr->relOffset )
 *op16Ptr->jumpAddress = adjustData( op16Ptr, destination, source, fixedBytes,
   opPtr->relOffset );
if( opPtr->flagMask & NOENLARGE_FLAG )
  *op16Ptr->extra = -*op16Ptr->extra;
if( opPtr->flagMask & DYNAMIC_FLAG )
  *op16Ptr->jumpAddress = DYNAMIC_X86INSTRUCTION;
return source + bytes;
}

PBYTE transferDataPrefix( X86_16BIT_INSTRUCTION* op16Ptr, X86INSTRUCTION* opPtr,
PBYTE destination, PBYTE source )
{
transferData( op16Ptr, opPtr, destination, source );
opPtr = &instructionMap[source[1]];
return opPtr->pXferFunction(op16Ptr, opPtr, destination + 1, source + 1);
}

PBYTE adjustData( X86_16BIT_INSTRUCTION* op16Ptr, PBYTE destination, PBYTE source,
LONG bytes, LONG targetOffset )
{
LONG oldOffset = 0;
LONG newOffset = 0;
PBYTE target;
LONG targetSize = bytes - targetOffset;
PVOID targetAddr = &destination[targetOffset];
switch( targetSize )
{
  case 1:
   oldOffset = (LONG)*((PCHAR)targetAddr);
   *op16Ptr->extra = 3;
   break;
  case 2:
   oldOffset = (LONG)*((PSHORT)targetAddr);
   *op16Ptr->extra = 2;
   break;
  case 4:
   oldOffset = (LONG)*((PLONG)targetAddr);
   *op16Ptr->extra = 0;
   break;
}
target = source + bytes + oldOffset;
newOffset = oldOffset - (destination - source);
switch( targetSize )
{
   case 1:
    *((PCHAR)targetAddr) = (CHAR)newOffset;
    break;
   case 2:
    *((PSHORT)targetAddr) = (SHORT)newOffset;
    break;
   case 4:
    *((PLONG)targetAddr) = (LONG)newOffset;
    break;
}
ASSERT( destination + bytes + newOffset == target );
return target;
}

PBYTE noTransferOp( X86_16BIT_INSTRUCTION* op16Ptr, X86INSTRUCTION* opPtr, PBYTE
destination, PBYTE source )
{
return source + 1;
UNREFERENCED_PARAMETER( destination );
UNREFERENCED_PARAMETER( opPtr );
UNREFERENCED_PARAMETER( op16Ptr );
}

PBYTE transferOp0F( X86_16BIT_INSTRUCTION* op16Ptr, X86INSTRUCTION* opPtr, PBYTE
destination, PBYTE source )
{
transferData( op16Ptr, opPtr, destination, source );
opPtr = &extendedInstructionMap[source[1]];
return opPtr->pXferFunction( op16Ptr, opPtr, destination + 1, source + 1 );
}

PBYTE transferOp66( X86_16BIT_INSTRUCTION* op16Ptr, X86INSTRUCTION* opPtr, PBYTE
destination, PBYTE source )
{
op16Ptr->operandIs16 = 1;
return transferDataPrefix( op16Ptr, opPtr, destination, source );
}

PBYTE transferOp67( X86_16BIT_INSTRUCTION* op16Ptr, X86INSTRUCTION* opPtr, PBYTE
destination, PBYTE source )
{
op16Ptr->addressIs16 = 1;
return transferDataPrefix( op16Ptr, opPtr, destination, source );
}

PBYTE transferOpF6( X86_16BIT_INSTRUCTION* op16Ptr, X86INSTRUCTION* opPtr, PBYTE
destination, PBYTE source )
{
if( (source[1] & 0x38) == 0x00 )
{
  X86INSTRUCTION ce = { 0xf6, transfer2Mod1 };
  return ce.pXferFunction( op16Ptr, &ce, destination, source );
}
{
  X86INSTRUCTION ce = { 0xf6, transfer2Mod };
  return ce.pXferFunction( op16Ptr, &ce, destination, source );
}
}

PBYTE transferOpF7( X86_16BIT_INSTRUCTION* op16Ptr, X86INSTRUCTION* opPtr, PBYTE
destination, PBYTE source )
{
if( (source[1] & 0x38) == 0x00 )
{
  X86INSTRUCTION ce = { 0xf7, transfer2ModOperand };
  return ce.pXferFunction( op16Ptr, &ce, destination, source );
}
{
  X86INSTRUCTION ce = { 0xf7, transfer2Mod };
  return ce.pXferFunction( op16Ptr, &ce, destination, source );
}
}

PBYTE transferOpFF( X86_16BIT_INSTRUCTION* op16Ptr, PX86INSTRUCTION opPtr, PBYTE
destination, PBYTE source )
{
if( source[1] == 0x15 || source[1] == 0x25 )
{
  PBYTE* jumpAddress = *(PBYTE**) &source[2];
  *op16Ptr->jumpAddress = *jumpAddress;
}
else if( (source[1] & 0x38) == 0x10 || (source[1] & 0x38) == 0x18 ||
   (source[1] & 0x38) == 0x20 || (source[1] & 0x38) == 0x28 )
{
  *op16Ptr->jumpAddress = DYNAMIC_X86INSTRUCTION;
}
{
  X86INSTRUCTION ce = { 0xff, transfer2Mod };
  return ce.pXferFunction( op16Ptr, &ce, destination, source );
}
}

//called by isJump when getx86Instruction wasn't enough to determine type
ULONG getNextInstruction( PCHAR codePtr, ULONG initial, PCHAR destinationBuffer,
ULONG destinationBufferLength )
{
PBYTE source = NULL;
PBYTE destination = NULL;
ULONG bytesCopied = 0;
PBYTE target = NULL;
LONG  extra = 0;
memset( destinationBuffer, 0, destinationBufferLength );
source = (PBYTE)codePtr;
destination = (PBYTE)destinationBuffer;
for( bytesCopied = 0; bytesCopied < initial; )
{
  source = transferInstruction( destination, source, &target, &extra );
  if( !source )
  {
   memset( destinationBuffer, 0, destinationBufferLength );
   bytesCopied = 0;
   break;
  }
```
