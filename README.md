'use client'

import { useState, useRef, useEffect } from 'react'
import QrScanner from 'react-qr-scanner'
import SignatureCanvas from 'react-signature-canvas'
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import { Label } from "@/components/ui/label"
import { Alert, AlertDescription } from "@/components/ui/alert"
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogTrigger } from "@/components/ui/dialog"
import { format, parse, isWithinInterval } from 'date-fns'

export default function LunchQRApp() {
  const [isCameraActive, setIsCameraActive] = useState(false)
  const [scannedCode, setScannedCode] = useState('')
  const [scanDate, setScanDate] = useState('')
  const [lunchQuantity, setLunchQuantity] = useState(0)
  const [facingMode, setFacingMode] = useState('environment')
  const [scanError, setScanError] = useState(null)
  const [registeredData, setRegisteredData] = useState([])
  const [startDate, setStartDate] = useState('')
  const [endDate, setEndDate] = useState('')
  const [filteredData, setFilteredData] = useState([])
  const signatureRef = useRef()

  const handleScan = (data) => {
    if (data) {
      setScannedCode(data.text)
      setScanDate(format(new Date(), 'yyyy-MM-dd HH:mm:ss'))
      setIsCameraActive(false)
      setScanError(null)
    }
  }

  const handleError = (err) => {
    console.error(err)
    setScanError(err.message)
  }

  const toggleCamera = () => {
    setIsCameraActive(!isCameraActive)
    setScanError(null)
  }

  const toggleFacingMode = () => {
    setFacingMode(prevMode => prevMode === 'environment' ? 'user' : 'environment')
  }

  const handleQuantityChange = (e) => {
    const value = parseInt(e.target.value)
    setLunchQuantity(isNaN(value) ? 0 : value)
  }

  const handleSubmit = () => {
    if (scannedCode && !signatureRef.current.isEmpty() && lunchQuantity > 0) {
      const newData = {
        code: scannedCode,
        date: scanDate,
        quantity: lunchQuantity,
        signature: signatureRef.current.toDataURL()
      }
      setRegisteredData([...registeredData, newData])
      // Reset form
      setScannedCode('')
      setScanDate('')
      setLunchQuantity(0)
      signatureRef.current.clear()
    } else {
      alert('Por favor, escanee un código QR, ingrese una cantidad válida de almuerzos (mínimo 1) y firme antes de registrar.')
    }
  }

  const filterDataByDateRange = () => {
    if (!startDate || !endDate) {
      setFilteredData(registeredData)
      return
    }

    const filteredResults = registeredData.filter(data => {
      const dataDate = parse(data.date, 'yyyy-MM-dd HH:mm:ss', new Date())
      const start = parse(startDate, 'yyyy-MM-dd', new Date())
      const end = parse(endDate, 'yyyy-MM-dd', new Date())
      return isWithinInterval(dataDate, { start, end })
    })

    setFilteredData(filteredResults)
  }

  useEffect(() => {
    filterDataByDateRange()
  }, [registeredData, startDate, endDate])

  return (
    <div className="container mx-auto p-4 max-w-md">
      <h1 className="text-2xl font-bold mb-4">Registro de Almuerzos</h1>
      
      <div className="mb-4 space-x-2">
        <Button onClick={toggleCamera}>
          {isCameraActive ? 'Desactivar Cámara' : 'Activar Cámara'}
        </Button>
        <Button onClick={toggleFacingMode}>
          Cambiar Cámara
        </Button>
      </div>
      
      {isCameraActive && (
        <div className="mb-4">
          <QrScanner
            delay={300}
            onError={handleError}
            onScan={handleScan}
            style={{ width: '100%' }}
            constraints={{
              audio: false,
              video: { facingMode: facingMode }
            }}
          />
          <Alert className="mt-2">
            <AlertDescription>
              Cámara activa. Buscando código QR...
            </AlertDescription>
          </Alert>
        </div>
      )}
      
      {scanError && (
        <Alert variant="destructive" className="mb-4">
          <AlertDescription>
            Error: {scanError}
          </AlertDescription>
        </Alert>
      )}
      
      {scannedCode && (
        <div className="mb-4">
          <p>Código Escaneado: {scannedCode}</p>
          <p>Fecha de Escaneo: {scanDate}</p>
        </div>
      )}
      
      <div className="mb-4">
        <Label htmlFor="lunch-quantity" className="block mb-2">
          Cantidad de Almuerzos:
        </Label>
        <Input
          id="lunch-quantity"
          type="number"
          value={lunchQuantity}
          onChange={handleQuantityChange}
          min="0"
          className="w-full"
        />
      </div>
      
      <div className="mb-4">
        <Label htmlFor="signature" className="block mb-2">
          Firma:
        </Label>
        <SignatureCanvas
          ref={signatureRef}
          canvasProps={{
            className: 'border w-full h-40',
          }}
        />
        <Button onClick={() => signatureRef.current.clear()} className="mt-2">
          Limpiar Firma
        </Button>
      </div>
      
      <Button onClick={handleSubmit} className="w-full mb-4">
        Registrar Almuerzo
      </Button>
      
      <Dialog>
        <DialogTrigger asChild>
          <Button variant="outline" className="w-full">
            Ver Datos Registrados
          </Button>
        </DialogTrigger>
        <DialogContent className="max-w-3xl max-h-[80vh] overflow-y-auto">
          <DialogHeader>
            <DialogTitle>Datos Registrados</DialogTitle>
          </DialogHeader>
          <div className="grid gap-4">
            <div className="flex space-x-2">
              <div>
                <Label htmlFor="start-date">Fecha Inicio:</Label>
                <Input
                  id="start-date"
                  type="date"
                  value={startDate}
                  onChange={(e) => setStartDate(e.target.value)}
                />
              </div>
              <div>
                <Label htmlFor="end-date">Fecha Fin:</Label>
                <Input
                  id="end-date"
                  type="date"
                  value={endDate}
                  onChange={(e) => setEndDate(e.target.value)}
                />
              </div>
            </div>
            {filteredData.map((data, index) => (
              <div key={index} className="border p-4 rounded">
                <p>Código: {data.code}</p>
                <p>Fecha: {data.date}</p>
                <p>Cantidad: {data.quantity}</p>
                <img src={data.signature} alt="Firma" className="mt-2 border" />
              </div>
            ))}
          </div>
        </DialogContent>
      </Dialog>
    </div>
  )
}
