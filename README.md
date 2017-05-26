# expert-spoon
 UChar*        getHeight             ()                        { return m_puhHeight;         }
   UChar         getHeight             ( UInt uiIdx )            { return m_puhHeight[uiIdx];  }



  UChar*        getWidth              ()                        { return m_puhWidth;          }
   UChar         getWidth              ( UInt uiIdx )            { return m_puhWidth[uiIdx];   }

 Void TComDataCU::getPartIndexAndSize( UInt uiPartIdx, UInt& ruiPartAddr, Int& riWidth, Int& riHeight )
 {
   switch ( m_pePartSize[0] )
   {
     case SIZE_2NxN:
       riWidth = getWidth(0);      riHeight = getHeight(0) >> 1; ruiPartAddr = ( uiPartIdx == 0 )? 0 : m_uiNumPartition >> 1;
       break;
     case SIZE_Nx2N:
       riWidth = getWidth(0) >> 1; riHeight = getHeight(0);      ruiPartAddr = ( uiPartIdx == 0 )? 0 : m_uiNumPartition >> 2;
       break;
     case SIZE_NxN:
       riWidth = getWidth(0) >> 1; riHeight = getHeight(0) >> 1; ruiPartAddr = ( m_uiNumPartition >> 2 ) * uiPartIdx;
       break;
     case SIZE_2NxnU:
       riWidth     = getWidth(0);
       riHeight    = ( uiPartIdx == 0 ) ?  getHeight(0) >> 2 : ( getHeight(0) >> 2 ) + ( getHeight(0) >> 1 );
       ruiPartAddr = ( uiPartIdx == 0 ) ? 0 : m_uiNumPartition >> 3;
       break;
     case SIZE_2NxnD:
       riWidth     = getWidth(0);
       riHeight    = ( uiPartIdx == 0 ) ?  ( getHeight(0) >> 2 ) + ( getHeight(0) >> 1 ) : getHeight(0) >> 2;
       ruiPartAddr = ( uiPartIdx == 0 ) ? 0 : (m_uiNumPartition >> 1) + (m_uiNumPartition >> 3);
       break;
     case SIZE_nLx2N:
       riWidth     = ( uiPartIdx == 0 ) ? getWidth(0) >> 2 : ( getWidth(0) >> 2 ) + ( getWidth(0) >> 1 );
       riHeight    = getHeight(0);
       ruiPartAddr = ( uiPartIdx == 0 ) ? 0 : m_uiNumPartition >> 4;
       break;
     case SIZE_nRx2N:
       riWidth     = ( uiPartIdx == 0 ) ? ( getWidth(0) >> 2 ) + ( getWidth(0) >> 1 ) : getWidth(0) >> 2;
       riHeight    = getHeight(0);
       ruiPartAddr = ( uiPartIdx == 0 ) ? 0 : (m_uiNumPartition >> 2) + (m_uiNumPartition >> 4);
       break;
     default:
       assert ( m_pePartSize[0] == SIZE_2Nx2N );
       riWidth = getWidth(0);      riHeight = getHeight(0);      ruiPartAddr = 0;
       break;
   }
 }


  TComSlice*       getSlice           ()                        { return m_pcSlice;         }
   const TComSlice* getSlice           () const                  { return m_pcSlice;         }


   UInt&         getCtuRsAddr          ()                        { return m_ctuRsAddr;       }
   UInt          getCtuRsAddr          () const                  { return m_ctuRsAddr;       }


  UInt          getZorderIdxInCtu     () const                  { return m_absZIdxInCtu;    }



  Bool*         getCUTransquantBypass ()                        { return m_CUTransquantBypass;        }
   Bool          getCUTransquantBypass( UInt uiIdx )             { return m_CUTransquantBypass[uiIdx]; }
 

 Char*         getPartitionSize      ()                        { return m_pePartSize;        }
   PartSize      getPartitionSize      ( UInt uiIdx )            { return static_cast<PartSize>( m_pePartSize[uiIdx] ); }


 UChar*        getDepth              ()                        { return m_puhDepth;        }
   UChar         getDepth              ( UInt uiIdx ) const      { return m_puhDepth[uiIdx]; }


Void TComDataCU::clipMv    (TComMv&  rcMv)
 {
   Int  iMvShift = 2;
   Int iOffset = 8;
   Int iHorMax = ( m_pcSlice->getSPS()->getPicWidthInLumaSamples() + iOffset - m_uiCUPelX - 1 ) << iMvShift;
   Int iHorMin = (       -(Int)g_uiMaxCUWidth - iOffset - (Int)m_uiCUPelX + 1 ) << iMvShift;
 
   Int iVerMax = ( m_pcSlice->getSPS()->getPicHeightInLumaSamples() + iOffset - m_uiCUPelY - 1 ) << iMvShift;
   Int iVerMin = (       -(Int)g_uiMaxCUHeight - iOffset - (Int)m_uiCUPelY + 1 ) << iMvShift;
 
   rcMv.setHor( min (iHorMax, max (iHorMin, rcMv.getHor())) );
   rcMv.setVer( min (iVerMax, max (iVerMin, rcMv.getVer())) );
 }
 




/*TComYuv*/

 Void TComYuv::copyPartToPartYuv   ( TComYuv* pcYuvDst, const UInt uiPartIdx, const UInt iWidth, const UInt iHeight ) const
 {
   for(Int ch=0; ch<getNumberValidComponents(); ch++)
   {
     copyPartToPartComponent   (ComponentID(ch), pcYuvDst, uiPartIdx, iWidth>>getComponentScaleX(ComponentID(ch)), iHeight>>getComponentScaleY(ComponentID(ch)) );
   }
 }
 

 Void TComYuv::copyPartToPartComponent  ( const ComponentID ch, TComYuv* pcYuvDst, const UInt uiPartIdx, const UInt iWidthComponent, const UInt iHeightComponent ) const
 {
   const Pel* pSrc =           getAddr(ch, uiPartIdx);
         Pel* pDst = pcYuvDst->getAddr(ch, uiPartIdx);
   if( pSrc == pDst )
   {
     //th not a good idea
     //th best would be to fix the caller
     return ;
   }
 
   const UInt  iSrcStride = getStride(ch);
   const UInt  iDstStride = pcYuvDst->getStride(ch);
   for ( UInt y = iHeightComponent; y != 0; y-- )
   {
     ::memcpy( pDst, pSrc, iWidthComponent * sizeof(Pel) );
     pSrc += iSrcStride;
     pDst += iDstStride;
   }
 }
 

 Void TComYuv::removeHighFreq( const TComYuv* pcYuvSrc, const UInt uiPartIdx, const UInt uiWidth, UInt const uiHeight )
 {
   for(Int chan=0; chan<getNumberValidComponents(); chan++)
   {
     const ComponentID ch=ComponentID(chan);
 #if !DISABLING_CLIP_FOR_BIPREDME
     const ChannelType chType=toChannelType(ch);
 #endif
 
     const Pel* pSrc  = pcYuvSrc->getAddr(ch, uiPartIdx);
     Pel* pDst  = getAddr(ch, uiPartIdx);
 
     const Int iSrcStride = pcYuvSrc->getStride(ch);
     const Int iDstStride = getStride(ch);
     const Int iWidth  = uiWidth >>getComponentScaleX(ch);
     const Int iHeight = uiHeight>>getComponentScaleY(ch);
 
     for ( Int y = iHeight-1; y >= 0; y-- )
     {
       for ( Int x = iWidth-1; x >= 0; x-- )
       {
 #if DISABLING_CLIP_FOR_BIPREDME
         pDst[x ] = (2 * pDst[x]) - pSrc[x];
 #else
         pDst[x ] = Clip((2 * pDst[x]) - pSrc[x], chType);
 #endif
       }
       pSrc += iSrcStride;
       pDst += iDstStride;
     }
   }
 }
 


 Pel*         getAddr                    (const ComponentID id)                    { return m_apiBuf[id]; }
   const Pel*   getAddr                    (const ComponentID id) const              { return m_apiBuf[id]; }
 
   //  Access starting position of YUV partition unit buffer
   Pel*         getAddr                    (const ComponentID id, const UInt uiPartUnitIdx)
                                               {
                                                   Int blkX = g_auiRasterToPelX[ g_auiZscanToRaster[ uiPartUnitIdx ] ] >> getComponentScaleX(id);
                                                   Int blkY = g_auiRasterToPelY[ g_auiZscanToRaster[ uiPartUnitIdx ] ] >> getComponentScaleY(id);
                                                   assert((blkX<getWidth(id) && blkY<getHeight(id)));
                                                   return m_apiBuf[id] + blkX + blkY * getStride(id);
                                               }
   const Pel*   getAddr                    (const ComponentID id, const UInt uiPartUnitIdx) const
                                               {
                                                   Int blkX = g_auiRasterToPelX[ g_auiZscanToRaster[ uiPartUnitIdx ] ] >> getComponentScaleX(id);
                                                   Int blkY = g_auiRasterToPelY[ g_auiZscanToRaster[ uiPartUnitIdx ] ] >> getComponentScaleY(id);
                                                   assert((blkX<getWidth(id) && blkY<getHeight(id)));
                                                   return m_apiBuf[id] + blkX + blkY * getStride(id);
                                               }
 
   //  Access starting position of YUV transform unit buffer
   Pel*         getAddr                    (const ComponentID id, const UInt iTransUnitIdx, const UInt iBlkSizeForComponent)
                                               {
                                                 UInt width=getWidth(id);
                                                 Int blkX = ( iTransUnitIdx * iBlkSizeForComponent ) &  ( width - 1 );
                                                 Int blkY = ( iTransUnitIdx * iBlkSizeForComponent ) &~ ( width - 1 );
                                                 if (m_chromaFormatIDC==CHROMA_422 && id!=COMPONENT_Y)
                                                 {
                                                   blkY<<=1;
                                                 }
                                                 return m_apiBuf[id] + blkX + blkY * iBlkSizeForComponent;
                                               }
 
   const Pel*   getAddr                    (const ComponentID id, const UInt iTransUnitIdx, const UInt iBlkSizeForComponent) const
                                               {
                                                 UInt width=getWidth(id);
                                                 Int blkX = ( iTransUnitIdx * iBlkSizeForComponent ) &  ( width - 1 );
                                                 Int blkY = ( iTransUnitIdx * iBlkSizeForComponent ) &~ ( width - 1 );
                                                 if (m_chromaFormatIDC==CHROMA_422 && id!=COMPONENT_Y)
                                                 {
                                                   blkY<<=1;
                                                 }
                                                 return m_apiBuf[id] + blkX + blkY * iBlkSizeForComponent;
                                               }



  UInt         getStride                  (const ComponentID id) const { return m_iWidth >> getComponentScaleX(id);   }




